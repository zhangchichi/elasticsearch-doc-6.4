# SQL CLI

Elasticsearch附带一个脚本，用于在其bin目录中运行SQL CLI：

```
$ ./bin/elasticsearch-sql-cli
```

包含SQL CLI的jar是一个独立的Java应用程序，脚本只是启动它。 您可以将其移动到其他计算机，而无需在其上安装Elasticsearch。

您可以将Elasticsearch实例的URL作为第一个参数传递给连接：

```
$ ./bin/elasticsearch-sql-cli https://some.server:9200
```

CLI运行后，您可以使用Elasticsearch支持的任何[查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-spec.html)：

```
sql> SELECT * FROM library WHERE page_count > 500 ORDER BY page_count DESC;
     author      |        name        |  page_count   | release_date
-----------------+--------------------+---------------+---------------
Peter F. Hamilton|Pandora's Star      |768            |1078185600000
Vernor Vinge     |A Fire Upon the Deep|613            |707356800000
Frank Herbert    |Dune                |604            |-144720000000
Alastair Reynolds|Revelation Space    |585            |953078400000
James S.A. Corey |Leviathan Wakes     |561            |1306972800000
```



