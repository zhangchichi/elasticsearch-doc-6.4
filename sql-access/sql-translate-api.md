# SQL翻译API

SQL 翻译API接受JSON文档中的SQL并将其转换为Elasticsearch本地查询。 例如：

```
POST /_xpack/sql/translate
{
    "query": "SELECT * FROM library ORDER BY page_count DESC",
    "fetch_size": 10
}
```

返回：

```
{
    "size" : 10,
    "docvalue_fields" : [
        {
            "field": "page_count",
            "format": "use_field_mapping"
        },
        {
            "field": "release_date",
            "format": "epoch_millis"
        }
    ],
    "_source": {
        "includes": [
            "author",
            "name"
        ],
        "excludes": []
    },
    "sort" : [
        {
            "page_count" : {
                "order" : "desc"
            }
        }
    ]
}
```

SQL在运行后提供结果集，在这种情况下，SQL会使用[scroll](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html) API.如果结果包含聚合，SQL会使用[普通查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)API.

请求体接受所有的 [SQL REST API](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-rest.html)  [ 参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-rest.html#sql-rest-fields)，除了 `cursor。`



