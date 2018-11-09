### Bulk API

Bulk API使得在单个API调用中执行许多索引/删除操作成为可能。 这可以大大提高索引速度。

> Bulk 的客户端支持
>
> 一些官方支持的客户端提供帮助程序来协助批量请求并将文档从一个索引重新索引到另一个索引：
>
> Perl 查看 [Search::Elasticsearch::Client::5\_0::Bulk](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Bulk) and [Search::Elasticsearch::Client::5\_0::Scroll](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Scroll)
>
> Python 查看 [elasticsearch.helpers.\*](http://elasticsearch-py.readthedocs.org/en/master/helpers.html)

REST API端点是 /\_bulk，它期望以下换行符分隔JSON（NDJSON）结构：

```
action_and_meta_data\n
optional_source\n
action_and_meta_data\n
optional_source\n
....
action_and_meta_data\n
optional_source\n
```

**备注：**最后一行数据必须以换行符`\n`结尾。 每个换行符前面都可以有回车符`\r\n`。 向此端点发送请求时，`Content-Type`标头应设置为`application / x-ndjson`。

可能的操作是`index`，`create`，`delete`和`update`。 `index`和`create`期望下一行的源，并且与标准索引API的`op_type`参数具有相同的语义（即如果已存在具有相同索引和类型的文档，则`create`将失败，而`index`将可能添加或替换文档）。 `delete`不期望以下行中的source，并且具有与标准删除API相同的语义。 `update`期望在下一行指定部分`doc`，`upsert`和`script`及其选项。

如果要为curl提供文本文件输入，则必须使用--data-binary标志而不是plain -d。 后者不保留换行符。 例：

```
$ cat requests
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
$ curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "@requests"; echo
{"took":7, "errors": false, "items":[{"index":{"_index":"test","_type":"_doc","_id":"1","_version":1,"result":"created","forced_refresh":false}}]}
```

因为这种格式使用`\n`作为分隔符，所以请确保JSON操作和源不要实用`petty`。 以下是批量命令的正确序列示例：

```
POST _bulk
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

操作结果：

```
{
   "took": 30,
   "errors": false,
   "items": [
      {
         "index": {
            "_index": "test",
            "_type": "_doc",
            "_id": "1",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 0,
            "_primary_term": 1
         }
      },
      {
         "delete": {
            "_index": "test",
            "_type": "_doc",
            "_id": "2",
            "_version": 1,
            "result": "not_found",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 404,
            "_seq_no" : 1,
            "_primary_term" : 2
         }
      },
      {
         "create": {
            "_index": "test",
            "_type": "_doc",
            "_id": "3",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 2,
            "_primary_term" : 3
         }
      },
      {
         "update": {
            "_index": "test",
            "_type": "_doc",
            "_id": "1",
            "_version": 2,
            "result": "updated",
            "_shards": {
                "total": 2,
                "successful": 1,
                "failed": 0
            },
            "status": 200,
            "_seq_no" : 3,
            "_primary_term" : 4
         }
      }
   ]
}
```

端点是/\_bulk，/{index} /\_ bulk和{index} /{type} /\_bulk。 提供索引或索引/类型时，默认情况下将对未明确提供它们的批量项使用它们。

关于格式的说明。 这里的想法是尽可能快地处理这个。 由于某些操作将被重定向到其他节点上的其他分片，因此在接收节点侧仅解析`action_meta_data`。

使用此协议的客户端库应尝试在客户端尝试执行类似操作，并尽可能减少缓冲。

对批量操作的响应是一个大型JSON结构，其中包含已执行的每个操作的单独结果。 单个操作的失败不会影响剩余的操作。

在单个bulk调用中没有“正确”的操作数量。 您应该尝试不同的设置，以找到特定工作负载的最佳大小。

如果使用HTTP API，请确保客户端不发送HTTP块（http chunks），因为这会降低速度。

### 版本

每个批量项目都可以使用版本字段包含版本值。 它会根据\_version映射自动跟踪index/delete操作的行为。 它还支持`version_type`（参见[版本控制](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-versioning)）

### 路由

每个批量项目都可以使用`routing`字段包含路由值。 它会根据`_routing`映射自动遵循index / delete操作的行为。

### 等待活动分片

进行批量调用时，可以将`wait_for_active_shards`参数设置为在开始处理批量请求之前要求最少数量的分片副本处于活动状态。 有关详细信息和用法示例，请参见[此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards)。

### 刷新（refresh）

控制何时此请求所做的更改对搜索可见。 请参阅[刷新](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)。

### 更新（update）

使用更新操作时，retry\_on\_conflict可用作操作本身的字段（不在额外的有效负载行中），以指定在版本冲突的情况下应重试更新的次数。

更新操作有效内容支持以下选项：doc（部分文档），upsert，doc\_as\_upsert，script，params（用于脚本），lang（用于脚本）和\_source。 有关选项的详细信息，请参阅更新文档。 更新操作的示例：

```
POST _bulk
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"} }
{ "update" : { "_id" : "0", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "script" : { "source": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}
{ "update" : {"_id" : "2", "_type" : "_doc", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"}, "doc_as_upsert" : true }
{ "update" : {"_id" : "3", "_type" : "_doc", "_index" : "index1", "_source" : true} }
{ "doc" : {"field" : "value"} }
{ "update" : {"_id" : "4", "_type" : "_doc", "_index" : "index1"} }
{ "doc" : {"field" : "value"}, "_source": true}
```

### 安全

查看 [_URL-based access control_](https://www.elastic.co/guide/en/elasticsearch/reference/current/url-access-control.html)



