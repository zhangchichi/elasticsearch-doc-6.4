# GET API

get API允许根据其id从索引中获取类型化的JSON文档。 以下示例从名为twitter的索引获取JSON文档，该索引类型为\_doc，id值为0：

```
GET twitter/_doc/0
```

结果：

```
{
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "0",
    "_version" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

上面的结果包括我们希望检索的文档的\_index，\_type，\_id和\_version，包括文档的实际\_source（如果可以找到）（如响应中的found字段所示）。

API还允许使用HEAD检查文档是否存在，例如：

```
HEAD twitter/_doc/0
```

### 实时

默认情况下，get API是实时的，并且不受索引刷新率的影响（当数据对搜索可见时）。 如果文档已更新但尚未刷新，则get API将就地发出刷新调用以使文档可见。 这也将使上次刷新后其他文档发生变化。 为了禁用实时GET，可以将realtime参数设置为false。

### Source过滤

默认情况下，除非已使用stored\_fields参数或禁用了\_source字段，否则get操作将返回\_source字段的内容。 您可以使用\_source参数关闭\_source检索：

```
GET twitter/_doc/0?_source=false
```

如果您只需要完整\_source中的一个或两个字段，则可以使用\_source\_include和\_source\_exclude参数来包含或过滤掉您需要的部分。 这对于大型文档尤其有用，其中部分检索可以节省网络开销。 这两个参数都使用逗号分隔的字段列表或通配符表达式。 例：

```
GET twitter/_doc/0?_source_include=*.id&_source_exclude=entities
```

如果您只想指定包含，则可以使用较短的表示法：

```
GET twitter/_doc/0?_source=*.id,retweeted
```

### 排序字段



