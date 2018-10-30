# 概览

Elasticsearch SQL旨在为Elasticsearch提供功能强大但轻量级的SQL接口。

### 介绍

Elasticsearch SQL是一个X-Pack组件，允许对Elasticsearch实时执行类似SQL的查询。无论是使用REST接口，命令行还是JDBC，任何客户端都可以使用SQL在Elasticsearch中本地搜索和聚合数据。人们可以将Elasticsearch SQL视为一个翻译器，它既能理解SQL又能理解Elasticsearch，并且通过利用Elasticsearch功能，可以实现大规模实时读取和处理数据。

### 为什么Elasticsearch SQL

**原生整合**

Elasticsearch SQL是为Elasticsearch构建的。根据底层存储，针对相关节点有效地执行每个查询。

**无外部依赖**

无需额外的硬件，进程，运行时或库来查询Elasticsearch; Elasticsearch SQL通过在Elasticsearch集群内运行来消除额外的移动部件。

**轻量级且高效**

Elasticsearch SQL不会抽象Elasticsearch及其搜索功能 - 相反，它包含并公开SQL以允许以相同的声明性，简洁的方式实时进行正确的全文搜索。

