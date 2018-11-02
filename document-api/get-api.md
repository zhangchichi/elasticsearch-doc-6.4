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

### 存储字段

get操作允许指定将通过传递stored\_fields参数返回的一组存储字段。 如果未存储请求的字段，则将忽略它们。 例如，考虑以下映射：

```
PUT twitter
{
   "mappings": {
      "_doc": {
         "properties": {
            "counter": {
               "type": "integer",
               "store": false
            },
            "tags": {
               "type": "keyword",
               "store": true
            }
         }
      }
   }
}
```

现在你可以添加一个文档：

```
PUT twitter/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

1. 试图检索它：

```
GET twitter/_doc/1?stored_fields=tags,counter
```

结果：

```
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "1",
   "_version": 1,
   "found": true,
   "fields": {
      "tags": [
         "red"
      ]
   }
}
```

从文档本身获取的字段值始终作为数组返回。 由于未存储`counter`字段，因此get请求在尝试获取stored\_field时会忽略它。

也可以检索像\_routing字段这样的元数据字段：

```
PUT twitter/_doc/2?routing=user1
{
    "counter" : 1,
    "tags" : ["white"]
}
```

```
GET twitter/_doc/2?routing=user1&stored_fields=tags,counter
```

结果：

```
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "2",
   "_version": 1,
   "_routing": "user1",
   "found": true,
   "fields": {
      "tags": [
         "white"
      ]
   }
}
```

### 直接获取\_source

使用/ {index} / {type} / {id} / \_ source端点只获取文档的\_source字段，而不包含任何其他内容。 例如：

```
GET twitter/_doc/1/_source
```

您还可以使用相同的源过滤参数来控制将返回\_source的哪些部分：

```
GET twitter/_doc/1/_source?_source_include=*.id&_source_exclude=entities'
```

注意，\_source端点还有一个HEAD变体，可以有效地测试document \_source的存在。 如果在映射中禁用了现有文档，则该文档将没有\_source。

```
HEAD twitter/_doc/1/_source
```

### 路由

使用控制路由的能力进行索引时，为了获取文档，还应提供路由值。 例如：

```
GET twitter/_doc/2?routing=user1
```

以上将获得id为2的`twitter`文档，但将根据用户进行路由。 注意，在没有正确路由的情况下发出get，将导致无法获取文档。

### 优先权

`preference`控制哪些分片副本执行get请求。 默认情况下，操作在分片复制副本之间随机化。

`preference能被设置为：`

```
_primary
    该操作将仅在主分片上执行。
_local
    如果可能，操作将优先在本地分配的分片上执行。
```

自定义值

自定义值将用于保证相同的分片将用于相同的自定义值。 当在不同的刷新状态下击中不同的分片时，这可以帮助“跳跃值”。 示例值可以是Web会话ID或用户名。

### 刷新

可以将refresh参数设置为true，以便在get操作之前刷新相关的分片并使其可搜索。 将其设置为true应该在仔细考虑并验证这不会对系统造成重负荷（并减慢索引）之后进行。

### 分布式

get操作被散列为特定的分片ID。 然后它被重定向到该分片ID中的一个副本并返回结果。 副本是主分片及其在该分片ID组中的副本。 这意味着我们拥有的副本越多，我们将获得更好的GET缩放。

### 版本支持

仅当版本参数的当前版本等于指定的版本时，才可以使用version参数来检索文档。 除了始终检索文档的版本类型FORCE之外，所有版本类型的此行为都是相同的。 请注意，不推荐使用FORCE版本类型。

在内部，Elasticsearch已将旧文档标记为已删除并添加了一个全新的文档。 旧版本的文档不会立即消失，但您将无法访问它。 当您继续索引更多数据时，Elasticsearch会在后台清除已删除的文档。



