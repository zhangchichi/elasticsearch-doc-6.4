# 跨SQL和Elasticsearch的概念映射

虽然SQL和Elasticsearch对数据的组织方式（已经不同的语义）有不同的术语，但基本上他们的目的是相同的。

那么就让我们从底部开始吧，他们大致是：

| sql | elasticsearch | description |
| :---: | :---: | :--- |
| column | field | 在这两种情况下，在最低界别，数据存储在包含一个值和一个[数据类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-data-types.html)的命名条目中。SQL把这个条目称为column而Elasticsearch把它叫做field。注意在Elasticsearch的一个feild中，可以包含多个类型相同的值（实际上是列表）但是在sql中只能包含所述类型的一个值。Elasticsearch会尽最大努力保留sql语义，并且根据查询拒绝返回多个值的feild的sql语义。 |
| row | document | columns 和 fields 本身不存在，他们是row或者document的一部分。这两者的语义略有不同：一行往往是严格的（并且具有更多强制），而文档往往更灵活或更松散（同时仍然具有结构）。 |
| schema | implicit | 在RDBMS中，schema主要用于表的命名空间，通常用作安全边界。Elasticsearch没有为此提供一个同等的概念。但是当启用安全性时，Elasticsearch会强制启动安全性，这样角色只能看到它被允许的数据。 |
| catlog or database | cluster instance | 在SQL中，catlog和database可以相互转换，并且表示一组schema\(即多个表\)。在Elasticsearch中一组可用的所用集合在一个cluster中。语义也有些不同，database本质上是另一个命名空间（可能对数据的存储方式有一些影响），而Elasticsearch集群是一个运行时实例，或者更确切地说是一组至少一个Elasticsearch实例（通常运行分布式）。实际上，这意味着在SQL中，一个实例中可能有多个catlog，在Elasticsearch中，一个仅限于一个。 |
| cluster | cluster\(联合\) | 传统上在SQL中，cluster指的是包含许多目录或数据库的单个RDMBS实例（参见上文）。 同样的单词也可以在Elasticsearch中重用，但是它的语义有点澄清。虽然RDBMS在一台机器（非分布式）上往往只有一个正在运行的实例，但Elasticsearch采用相反的方式，默认情况下是分布式和多实例。此外，Elasticsearch集群可以以联合方式连接到其他集群，因此集群意味着：单个集群::多个Elasticsearch实例通常分布在机器上，在同一名称空间内运行。 多个集群::多个集群，每个集群都有自己的名称空间，在联合设置中相互连接（[请参阅跨集群搜索](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html)）。 |

正如人们可以看到的那样，虽然概念之间的映射并不是一对一的，并且语义有些不同，但共同点比差异要多。 实际上，由于SQL声明性质，许多概念可以透明地跨越Elasticsearch，并且两者的术语可以在整个材料的其余部分中互换使用。

