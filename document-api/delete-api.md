# DELETE API

delete API允许根据其id从特定索引中删除JSON文档。 以下示例从名为twitter的索引中删除JSON文档，该索引类型为\_doc，ID为1：

```
DELETE /twitter/_doc/1
```

删除结果：

```
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 2,
    "_primary_term": 1,
    "_seq_no": 5,
    "result": "deleted"
}
```

### 版本

索引的每个文档都是版本化的。 删除文档时，可以指定版本以确保我们尝试删除的相关文档实际上已被删除，并且在此期间它没有更改。 对文档执行的每个写入操作（包括删除）都会导致其版本递增。 删除文档的版本号在删除后仍可短时间使用，以便控制并发操作。 已删除文档的版本保持可用的时间长度由`index.gc_deletes`索引设置确定，默认为60秒。

### 路由

使用控制路由的能力进行索引时，为了删除文档，还应提供路由值。 例如：

```
DELETE /twitter/_doc/1?routing=kimchy
```

以上将删除id为1的`twitter`索引，但将根据用户进行路由。 请注意，在没有正确路由的情况下发出删除将导致文档不被删除。

当`_routing`映射根据需要设置且未指定路由值时，delete api将抛出`RoutingMissingException`并拒绝该请求。

### 自动创建索引

如果使用[外部扩展控制版本](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)，则删除操作会自动创建索引（如果之前尚未创建索引）（请参阅创[建索引API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)以手动创建索引），并且还会自动为特定类型创建动态类型映射（如果它 之前尚未创建（请查看[put mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html)以手动创建类型映射）。

### 分布式

删除操作将散列为特定的分片ID。 然后它被重定向到该id组内的主分片，并复制（如果需要）到该id组内的分片副本。

### 等待活动分片

进行删除请求时，可以将`wait_for_active_shards`参数设置为在开始处理删除请求之前要求最少数量的分片副本处于活动状态。 有关详细信息和用法示例，请参见[此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards)。

### 刷新

此请求的改变被查询可见。查看[此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)

### 超时

执行删除操作时，分配用于执行删除操作的主分片可能不可用。 造成这种情况的一些原因可能是主分片当前正在从存储中恢复或正在进行重定位（relocation）。 默认情况下，删除操作将在主分片上等待最多1分钟，然后失败并响应错误。 `timeout`参数可用于显式指定等待的时间。 以下是将其设置为5分钟的示例：

```
DELETE /twitter/_doc/1?timeout=5m
```













