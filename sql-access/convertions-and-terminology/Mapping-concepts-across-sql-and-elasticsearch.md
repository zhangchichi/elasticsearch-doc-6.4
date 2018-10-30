# 跨SQL和Elasticsearch的概念映射

虽然SQL和Elasticsearch对数据的组织方式（已经不同的语义）有不同的术语，但基本上他们的目的是相同的。

那么就让我们从底部开始吧，他们大致是：

| sql | elasticsearch | description |
| :---: | :---: | :--- |
| column | field | 在这两种情况下，在最低界别，数据存储在包含一个值和一个[数据类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-data-types.html)的命名条目中。SQL把这个条目称为column而Elasticsearch把它叫做field。注意在Elasticsearch的一个feild中，可以包含多个类型相同的值（实际上是列表）但是在sql中只能包含所述类型的一个值。Elasticsearch会尽最大努力保留sql语义，并且根据查询拒绝返回多个值的feild的sql语义。 |
| row | document | columns 和 fields 本身不存在，他们是row或者document的一部分。这两者的语义略有不同：一行往往是严格的（并且具有更多强制），而文档往往更灵活或更松散（同时仍然具有结构）。 |
| schema | implicit | 在RDBMS中，schema主要用于表的命名空间，通常用作安全边界。Elasticsearch没有为此提供一个同等的概念。但是当启用安全性时，Elasticsearch会强制启动安全性，这样角色只能看到它被允许的数据。 |
|  |  |  |



