### Multi Get API

Multi GET API允许基于索引，类型（可选）和id（以及可能的路由）获取多个文档。 响应包括一个`docs`数组，其中所有获取的文档按顺序对应于原始的multi-get请求（如果特定get的失败，则包含此错误的对象将包含在响应中）。 成功获取的结构在结构上类似于get API提供的文档。

例子：

```
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

mget端点也可以用于索引（在这种情况下，它在主体中不是必需的）：

```
GET /test/_mget
{
    "docs" : [
        {
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

类型也是

```
GET /test/_doc/_mget
{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}
```

在这种情况下，可以直接使用ids元素来简化请求：

```
GET /test/_doc/_mget
{
    "ids" : ["1", "2"]
}
```

### source 过滤

默认情况下，将为每个文档（如果存储）返回\_source字段。 与get API类似，您能使用\_source参数检索\_source的一部分（或根本不检索）。 您还可以使用url参数\_source，\_source\_include＆\_source\_exclude来指定默认值，这些默认值将在没有按文档说明时使用。

```
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}
```

### 字段

可以在每个返回文档中指定特定的stored fields，就和Get API中的[stored\_fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#get-stored-fields) 一样，例如：

```
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "stored_fields" : ["field1", "field2"]
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2",
            "stored_fields" : ["field3", "field4"]
        }
    ]
}
```

或者，您可以将查询字符串中的stored\_fields参数指定为默认值，以应用于所有文档。

```
GET /test/_doc/_mget?stored_fields=field1,field2
{
    "docs" : [
        {
            "_id" : "1"                      //1
        },
        {
            "_id" : "2",                             //2
            "stored_fields" : ["field3", "field4"] 
        }
    ]
}

1.返回field1 和 field2
2.返回field3 和 field4
```

### 路由

指定路由参数：

```
GET /_mget?routing=key1
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1",
            "routing" : "key2"
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

在此示例中，文档test / \_doc / 2将从对应于路由密钥key1的分片中获取，但是文档test / \_doc / 1将从对应于路由密钥key2的分片中获取。

### 安全

查看 [_URL-based access control_](https://www.elastic.co/guide/en/elasticsearch/reference/current/url-access-control.html)



