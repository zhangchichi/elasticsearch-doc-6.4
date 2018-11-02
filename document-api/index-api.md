# 索引API

**重要：查看**[**删除映射类型**](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)**。**

索引API在特定索引中添加或更新类型化的JSON文档，使其可搜索。 以下示例将JSON文档插入到“twitter”索引下一个名为\_doc的类型中且id为1：

```
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上述索引操作的结果是：

```
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result" : "created"
}
```

\_shards标头提供有关索引操作的复制过程的信息。

* `total `- 指示应在其上执行索引操作的分片副本（主分片和副本分片）的数量。
* `successful`-表示索引操作成功的分片副本数。
* `failed `- 在副本分片上索引操作失败的情况下包含复制相关错误的数组。

在`successful`至少为1的情况下，索引操作成功。

> **备注**
>
> 索引操作成功返回时，可能不是所有的副本分片开始操作（默认情况下，只需要主分区，但可以更改此行为）。 在这种情况下，total将等于基于number\_of\_replicas设置的总分片数，并且success将等于启动的分片数（主分片加副本）。 如果没有失败，则失败将为0。

### 索引自创建

如果之前尚未创建索引（索引用于手动创建索引的[create index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)），则索引操作会自动创建索引，如果尚未创建类型映射，则还会自动为特定类型创建动态类型映射（查看 [put mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html)用于手动创建类型映射）。

映射本身非常灵活，无需架构。 新字段和对象将自动添加到指定类型的映射定义中。 有关映射定义的更多信息，请查看[映射](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)部分。

通过在所有节点的配置文件中将action.auto\_create\_index设置为false，可以禁用自动索引创建。 通过将index.mapper.dynamic设置为false，可以禁用自动映射创建。

自动索引创建可以包括基于模式的白/黑列表，例如，设置 action.auto\_create\_index to + aaa \*， - bbb \*，+ ccc \*， - \*（+表示允许， - 表示不允许）。

### 版本

每个索引文档都有一个版本号。 关联的版本号作为对索引API请求的响应的一部分返回。 索引API可选地允许在指定version参数时进行乐观并发控制。 这将控制要针对该操作执行的文档的版本。 用于版本控制的用例的一个很好的例子是执行事务性读取然后更新。 从最初读取的文档中指定版本可确保在此期间未发生任何更改（在读取时为了更新，建议将首选项设置为\_primary）。 例如：

```
PUT twitter/_doc/1?version=2
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
```

**备注：版本控制是完全实时的，并且不受搜索操作的近实时方面的影响。 如果未提供版本，则执行该操作而不进行任何版本检查。**

默认情况下，使用从1开始的内部版本控制，并随每次更新,删除而增加。 版本号也可以用外部值补充（例如，如果在数据库中维护）。 要启用此功能，应将`version_type`设置为`external`。 提供的值必须是数字，长值大于或等于0，小于约9.2e + 18。 使用外部版本类型时，系统会检查传递给索引请求的版本号是否大于当前存储文档的版本，而不是检查匹配的版本号。 如果为true，则将索引文档并使用新版本号。 如果提供的值小于或等于存储文档的版本号，则会发生版本冲突，索引操作将失败。

在上面解释的内部和外部版本类型旁边，Elasticsearch还支持特定用例的其他类型。 以下是不同版本类型及其语义的概述。

```
internal
    如果给定版本与存储文档的版本相同，则仅索引文档。
external or external_gt
    给定版本严格高于存储文档的版本或者没有现有文档，则索引文档。 给定版本将用作新版本，并将与新文档一起存储。 提供的版本必须是非负长号。
external_gte
    仅在给定版本等于或高于存储文档的版本时索引文档。 如果没有现有文档，操作也将成功。 给定版本将用作新版本，并将与新文档一起存储。 提供的版本必须是非负长号。
```

**备注：external\_gte版本类型适用于特殊用例，应谨慎使用。 如果使用不当，可能会导致数据丢失。 还有另一个选项force，它被弃用，因为它可能导致主分片和副本分片发散。**

### 操作类型

索引操作还接受可用于强制创建操作的op\_type，允许“put-if-absent”行为。 使用create时，如果索引中已存在该id的文档，则索引操作将失败。

以下是使用op\_type参数的示例：

```
PUT twitter/_doc/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

另一种指定create的方式，使用下面的u'r'l:

```
PUT twitter/_doc/1/_create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

### 自动ID生成

可以在不指定id的情况下执行索引操作。 在这种情况下，将自动生成id。 此外，op\_type将自动设置为create。 这是一个例子（注意使用POST而不是PUT）：

```
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上述操作结果：

```
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "W0tpsmIBdwcYyG50zbta",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result": "created"
}
```

### 路由

默认情况下，通过使用文档的id值的哈希来控制分片放置 - 或路由。 对于更明确的控制，可以使用路由参数在每个操作的基础上直接指定输入到路由器使用的散列函数的值。 例如：

```
POST twitter/_doc?routing=kimchy
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

在上面的示例中，“\_ doc”文档根据提供的路由参数路由到分片：“kimchy”。

在设置显式映射时，可以选择使用\_routing字段来指示索引操作从文档本身中提取路由值。 这确实加大一个文档解析过程的（非常小的）成本。 如果定义了\_routing映射并将其设置为必需，则如果未提供或提取路由值，则索引操作将失败。

### 分散式

索引操作根据其路由指向主分片（请参阅上面的“路由”部分），并在包含此分片的实际节点上执行。 主分片完成操作后，如果需要，更新将分发到适用的副本。

### 等待活跃的碎片

为了提高对系统写入的弹性，可以将索引操作配置为在继续操作之前等待一定数量的活动分片副本。 如果必需数量的活动分片副本不可用，则写入操作必须等待并重试，直到必需的分片副本已启动或发生超时。 默认情况下，写操作仅等待主分片在继续之前处于活动状态（即`wait_for_active_shards = 1`）。通过设置`index.write.wait_for_active_shards`，可以在索引设置中动态覆盖此默认值。要更改每个操作的此行为，可以使用`wait_for_active_shards`请求参数。

有效值是全部或任何正整数，直到索引中每个分片的已配置副本总数（number\_of\_replicas + 1）。 指定负值或大于分片副本数的数字将引发错误。

例如，假设我们有一个包含三个节点A，B和C的集群，我们创建一个索引，副本数设置为3（产生4个分片副本，比节点多一个副本）。如果我们尝试索引操作，默认情况下，操作只会确保每个分片的主副本在继续之前可用。这意味着即使B和C发生故障，并且A托管了主分片副本，索引操作仍将只使用一个数据副本。如果在请求3上设置了wait\_for\_active\_shards（并且所有3个节点都已启动），那么索引操作将需要3个活动的分片副本才能继续执行，因为群集中有3个活动节点，每个都有一个，因此需要满足这一要求碎片的副本。但是，如果我们将wait\_for\_active\_shards设置为all（或者4，它们是相同的），则索引操作将不会继续，因为我们没有在索引中激活每个分片的所有4个副本。除非在群集中启动新节点以承载碎片的第四个副本，否则操作将超时。

重要的是要注意，此设置大大降低了写入操作不写入所需数量的分片副本的可能性，但它并未完全消除这种可能性，因为此检查在写入操作开始之前发生。 一旦写入操作正在进行，复制在任何数量的分片副本上仍然可能失败，但仍然可以在主要副本上成功。 写操作响应的\_shards部分显示复制成功/失败的分片副本数。

```
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    }
}
```

### 更新

控制何时此请求所做的更改对搜索可见。 请[参阅刷新](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)。

### 空修改

使用索引api更新文档时，即使文档未更改，也始终会创建新版本的文档。 如果这是不可接受的，请使用`_update `api，并将`detect_noop`设置为true。 索引api上没有此选项，因为索引api不会获取旧源，也无法将其与新源进行比较。

### 超时

执行索引操作时，分配用于执行索引操作的主分片可能不可用。 造成这种情况的一些原因可能是主分片当前正在从网关恢复或正在进行重定位。 默认情况下，索引操作将在主分片上等待最多1分钟，然后失败并响应错误。 timeout参数可用于显式指定等待的时间。 以下是将其设置为5分钟的示例：

```
PUT twitter/_doc/1?timeout=5m
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```



