# SQL REST API

SQL REST API接受JSON文档中的SQL，执行它并返回结果。 例如：

```
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM library ORDER BY page_count DESC LIMIT 5"
}
```

结果：

```
     author      |        name        |  page_count   | release_date
-----------------+--------------------+---------------+------------------------
Peter F. Hamilton|Pandora's Star      |768            |2004-03-02T00:00:00.000Z
Vernor Vinge     |A Fire Upon the Deep|613            |1992-06-01T00:00:00.000Z
Frank Herbert    |Dune                |604            |1965-06-01T00:00:00.000Z
Alastair Reynolds|Revelation Space    |585            |2000-03-15T00:00:00.000Z
James S.A. Corey |Leviathan Wakes     |561            |2011-06-02T00:00:00.000Z
```

虽然text / plain格式对人类来说很好，但计算机更喜欢更有结构的东西。 你可以用以下格式替换格式的值：

```
-json 又名 application/json 
-yaml 又名 application/yaml 
-smile 又名 application/smile 
-cbor 又名 application/cbor 
-txt 又名 text/plain 
-csv 又名 text/csv 
-tsv 又名 text/tab-separated-values
```

或者，您可以将Accept HTTP标头设置为适当的媒体格式\(appropriate media format\)。 GET参数优先于标头。 如果两者都未指定，则以与请求相同的格式返回响应。

```
POST /_xpack/sql?format=json
{
    "query": "SELECT * FROM library ORDER BY page_count DESC",
    "fetch_size": 5
}
```

返回：

```
{
    "columns": [
        {"name": "author",       "type": "text"},
        {"name": "name",         "type": "text"},
        {"name": "page_count",   "type": "short"},
        {"name": "release_date", "type": "date"}
    ],
    "rows": [
        ["Peter F. Hamilton",  "Pandora's Star",       768, "2004-03-02T00:00:00.000Z"],
        ["Vernor Vinge",       "A Fire Upon the Deep", 613, "1992-06-01T00:00:00.000Z"],
        ["Frank Herbert",      "Dune",                 604, "1965-06-01T00:00:00.000Z"],
        ["Alastair Reynolds",  "Revelation Space",     585, "2000-03-15T00:00:00.000Z"],
        ["James S.A. Corey",   "Leviathan Wakes",      561, "2011-06-02T00:00:00.000Z"]
    ],
    "cursor": "sDXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAEWWWdrRlVfSS1TbDYtcW9lc1FJNmlYdw==:BAFmBmF1dGhvcgFmBG5hbWUBZgpwYWdlX2NvdW50AWYMcmVsZWFzZV9kYXRl+v///w8="
}
```

您可以通过发回光标字段继续下一页。 在文本格式的情况下，光标作为Cursor http标头返回。

```
POST /_xpack/sql?format=json
{
    "cursor": "sDXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAEWYUpOYklQMHhRUEtld3RsNnFtYU1hQQ==:BAFmBGRhdGUBZgVsaWtlcwFzB21lc3NhZ2UBZgR1c2Vy9f///w8="
}
```

可能返回：

```
{
    "rows" : [
        ["Dan Simmons",        "Hyperion",             482,  "1989-05-26T00:00:00.000Z"],
        ["Iain M. Banks",      "Consider Phlebas",     471,  "1987-04-23T00:00:00.000Z"],
        ["Neal Stephenson",    "Snow Crash",           470,  "1992-06-01T00:00:00.000Z"],
        ["Frank Herbert",      "God Emperor of Dune",  454,  "1981-05-28T00:00:00.000Z"],
        ["Frank Herbert",      "Children of Dune",     408,  "1976-04-21T00:00:00.000Z"]
    ],
    "cursor" : "sDXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAEWODRMaXBUaVlRN21iTlRyWHZWYUdrdw==:BAFmBmF1dGhvcgFmBG5hbWUBZgpwYWdlX2NvdW50AWYMcmVsZWFzZV9kYXRl9f///w8="
}
```

注意 `column `对象只在第一页中展示。

当结果中没有返回光标时，您已到达最后一页。 与Elasticsearch的[`scroll`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html)一样，SQL可以在Elasticsearch中保持状态以支持游标。 与`scroll`不同，接收最后一页足以保证清除Elasticsearch状态。

要提前清除状态，可以使用clear cursor命令：

```
POST /_xpack/sql/close
{
    "cursor": "sDXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAAEWYUpOYklQMHhRUEtld3RsNnFtYU1hQQ==:BAFmBGRhdGUBZgVsaWtlcwFzB21lc3NhZ2UBZgR1c2Vy9f///w8="
}
```

可能返回：

```
{
    "succeeded" : true
}
```

可以在标准的Elasticsearch query DSL中运行SQL,并通过指定 filter 参数来过滤运行结果。

```
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM library ORDER BY page_count DESC",
    "filter": {
        "range": {
            "page_count": {
                "gte" : 100,
                "lte" : 200
            }
        }
    },
    "fetch_size": 5
}
```

结果：

```
  author     |                name                |  page_count   | release_date
---------------+------------------------------------+---------------+------------------------
Douglas Adams  |The Hitchhiker's Guide to the Galaxy|180            |1979-10-12T00:00:00.000Z

```

除了`query`和`cursor`字段之外，请求还可以包含`fetch_size`和`time_zone`。 `fetch_size`是每个页面返回多少结果的提示。 SQL可能会选择返回更多或更少的结果。 `time_zone`是用于日期函数和日期解析的时区。 `time_zone`默认为`utc`，可以采用[此处](http://www.joda.org/joda-time/apidocs/org/joda/time/DateTimeZone.html)记录的任何值。

