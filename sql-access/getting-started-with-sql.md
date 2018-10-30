# 入门sql

要开始使用Elasticsearch SQL，请创建包含一些数据的索引以进行试验：

```
PUT /library/book/_bulk?refresh
{"index":{"_id": "Leviathan Wakes"}}
{"name": "Leviathan Wakes", "author": "James S.A. Corey", "release_date": "2011-06-02", "page_count": 561}
{"index":{"_id": "Hyperion"}}
{"name": "Hyperion", "author": "Dan Simmons", "release_date": "1989-05-26", "page_count": 482}
{"index":{"_id": "Dune"}}
{"name": "Dune", "author": "Frank Herbert", "release_date": "1965-06-01", "page_count": 604}
```

现在你就可以立刻使用 SQL REST API 执行sql 了：

```
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM library WHERE release_date < '2000-01-01'"
}
```

会返回下面的内容：

```
    author     |     name      |  page_count   | release_date
---------------+---------------+---------------+------------------------
Dan Simmons    |Hyperion       |482            |1989-05-26T00:00:00.000Z
Frank Herbert  |Dune           |604            |1965-06-01T00:00:00.000Z
```

也可以使用 SQL CLI 。 在x-pack的bin目录中有一个启动它的脚本：

```
$ ./bin/elasticsearch-sql-cli
```

你也可以在这里运行同一个查询：

```
sql> SELECT * FROM library WHERE release_date < '2000-01-01';
    author     |     name      |  page_count   | release_date
---------------+---------------+---------------+------------------------
Dan Simmons    |Hyperion       |482            |1989-05-26T00:00:00.000Z
Frank Herbert  |Dune           |604            |1965-06-01T00:00:00.000Z
```



