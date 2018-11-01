# 读取和写入文档

### 介绍

Elasticsearch中的每个索引都分为碎片，每个碎片可以有多个副本。 这些副本称为复制组，在添加或删除文档时必须保持同步。 如果我们不这样做，从一个副本中读取将导致与从另一个副本读取的结果截然不同。 保持分片副本同步并从中提供读取的过程就是我们所说的**数据复制模型**。

Elasticsearch的数据复制模型基于主备份模型，并在Microsoft Research的[PacificA](https://www.microsoft.com/en-us/research/publication/pacifica-replication-in-log-based-distributed-storage-systems/)论文中得到了很好的描述。该模型基于单一复制的副本组，该副本充当主分片（primary shard）。 其他的副本称为备份分片（replica shards）。主分片作为全部索引操作的主入口，它负责验证它们并确保它们是正确的。当主分片接受到一个索引操作请求，它还负责将操作复制到其他副本。

本节的目的是对Elasticsearch复制模型进行高级概述，并讨论它对写入和读取操作之间的各种交互的影响。

### 写模型

Elasticsearch中的每个索引操作首先使用[路由](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-routing)解析为复制组，通常基于文档ID。确定复制组后，操作将在内部转发到组的当前主分片。主分片负责验证操作并将其转发到其他副本。由于副本可以脱机，因此不需要将主副本复制到所有副本。相反，Elasticsearch维护应该接收操作的分片副本列表。此列表称为同步副本，由主节点维护。顾名思义，这些是“好”分片副本的集合，保证已经处理了已经向用户确认的所有索引和删除操作。主要负责维护此不变量，因此必须将所有操作复制到此集合中的每个副本。

主分片遵循以下基本流程：

1. 验证传入操作并在结构无效时拒绝它（例如：在数字字段中传入对象）
2. 在本地执行操作，即索引或删除相关文档。 这也将验证字段的内容并在需要时拒绝（例如：关键字值太长，无法在Lucene中进行索引）。
3. 将操作转发到当前同步副本集中的每个副本。 如果有多个副本，则这是并行完成的。
4. 一旦所有副本成功执行了操作并响应主服务器，主服务器就会确认成功完成对客户端的请求。

### 失败处理

在索引编制过程中可能会出现许多问题 - 磁盘可能会损坏，节点可能会相互断开连接，或者某些配置错误可能导致复制副本上的操作失败，尽管它在主服务器上成功。这些是罕见的，但主分片必须回应它们。在主分片本身发生故障的情况下，托管主分片服务器的节点将向主服务器（the master?）发送有关它的消息。索引操作将等待（默认情况下最多1分钟），以便主服务器将其中一个副本提升为新的主分片。然后，该操作将被转发到新的主分片处理。请注意，主服务器还会监控节点的运行状况，并可能决定主动降级主分片。当通过网络问题将拥有主分片的节点与群集隔离时，通常会发生这种情况。 有关详细信息，请参见[此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-replication.html#demoted-primary)。

一旦在主服务器上成功执行了操作，主服务器就必须在副本分片上执行它时处理潜在的故障。 这可能是由副本上的实际故障或由于网络问题导致操作无法到达副本（或阻止副本响应）引起的。所有这些都具有相同的最终结果：作为同步副本集的一部分的副本错过了即将被确认的操作。为了避免违反不变量，主分片向主服务器发送消息，请求从同步副本集中删除有问题的分片。只有在主节点确认删除了分片后，主分片才会确认操作。 请注意，主服务器还将指示另一个节点开始构建新的分片副本，以便将系统还原到正常状态。

在将操作转发到副本时，主分片将使用副本来验证它仍然是活动的主分片。 如果主要由于网络分区（或长GC）而被隔离，则它可能会在意识到它已被降级之前继续处理传入的索引操作。复制分片将拒绝来自陈旧主分片的操作。 当主分片收到来自副本的拒绝其请求的响应，因为它不再是主分片，那么它将联系主服务器并将知道它已被替换。 然后将操作路由到新主服务器。

#### 如果没有副本会怎么样？

这是一个有效的方案，可能由于索引配置或仅因为所有副本都已失败而发生。 在这种情况下，主分片处理操作而没有任何外部验证，这可能看起来有问题。 另一方面，主分片本身不能使其他分片失败，除非请求主服务器代表它执行此操作。 这意味着主节点知道主分片是唯一的好副本。 因此，我们保证主节点不会将任何其他（过时的）分片副本提升为新的主分区，并且任何索引到主分区的操作都不会丢失。 当然，由于此时我们只使用单个数据副本运行，因此物理硬件问题可能导致数据丢失。 请参阅[Wait For Active Shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards)以获取一些缓解选项。

### 读模型

Elasticsearch中的读取可以是ID非常轻量级的查找，也可以是具有复杂聚合的大量搜索请求，这些聚合会占用非常多的CPU能力。主备份模型的优点之一是它使所有分片副本保持一致（除了在运行中的操作）。 因此，单个同步副本足以提供读取请求。

当节点收到读取请求时，该节点负责将其转发到保存相关分片的节点，整理响应并响应客户端。 我们将该节点称为该请求的协调节点（coordinating node）。 基本流程如下：

1. 将读取请求解析为相关分片。 请注意，由于大多数搜索将被发送到一个或多个索引，因此它们通常需要从多个分片中读取，每个分片代表数据的不同子集。
2. 从分片复制组中选择每个相关分片的活动副本。 这可以是主副本或副本。 默认情况下，Elasticsearch将简单地在分片副本之间循环。
3. 将分片级读取请求发送到所选副本。
4. 结合结果并做出回应。 请注意，在通过ID查找的情况下，只有一个分片是相关的，并且可以跳过此步骤。

#### 失败处理

当分片无法响应读取请求时，协调节点将从同一复制组中选择另一个副本，并将分片级别搜索请求发送到该副本。 重复失败可能导致没有可用的分片副本。 在某些情况下，例如\_search，Elasticsearch更愿意快速响应，尽管有部分结果，而不是等待问题得到解决（部分结果在响应的\_shards标头中指出）。

### 一些简单的含义

这些基本流程中的每一个都确定了Elasticsearch如何作为读取和写入系统的行为。 此外，由于读取和写入请求可以同时执行，因此这两个基本流程彼此交互。 这有一些固有的含义：

**高效读取**

在正常操作下，对每个相关复制组执行一次读取操作。 只有在失败条件下，同一个分片的多个副本才会执行相同的搜索。

**读取未确认**

由于主要的第一个索引在本地然后复制请求，因此并发读取可能在确认之前已经看到了更改。

**默认2分复制**

此模型可以容错，同时只保留两个数据副本。 这与基于法定数量的系统形成对比，其中容错的最小副本数为3。

### 故障

在失败的情况下，以下是可能的：

**单个分片可以减慢索引速度：**

由于主服务器在每个操作期间等待同步副本集中的所有副本，因此单个慢速分片可能会降低整个复制组的速度。 这是我们为上述阅读效率支付的价格。 当然，单个慢速分片也会减慢已经路由到它的不幸搜索。

**脏读：**

隔离的主分片可以暴露无法识别的写入。 这是因为隔离的主服务器只有在向其副本发送请求或向主服务器发送请求时才会意识到它是隔离的。 此时，操作已经索引到主分片中，并且可以通过并发读取来读取。 Elasticsearch通过每秒ping一次主服务器（默认情况下）并在没有master知道的情况下拒绝索引操作来减轻这种风险。

### 冰山一角

本文档提供了Elasticsearch如何处理数据的高度概述。 当然，还有很多事情要发生。 主要术语，集群状态发布和主选举等都在保持系统正常运行方面发挥作用。 本文档也未涵盖已知和重要的错误（已关闭和打开）。 我们认识到[GitHub](https://github.com/elastic/elasticsearch/issues?q=label%3Aresiliency)很难跟上。 为了帮助人们掌握这些优势，我们在网站上维护了一个专门的[弹性页面](https://www.elastic.co/guide/en/elasticsearch/resiliency/current/index.html)。 我们强烈建议阅读它。

### 




















