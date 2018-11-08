# Updata by Query API

\_update\_by\_query的最简单用法只是对索引中的每个文档执行更新而不更改源。 这对于获取新属性或其他一些在线映射更改很有用。 这是API：

```
POST twitter/_update_by_query?conflicts=proceed
```

类似结果：

```
{
  "took" : 147,
  "timed_out": false,
  "updated": 120,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 120,
  "failures" : [ ]
}
```

`_update_by_query`在索引启动时获取索引的快照，并使用内部版本控制索引它的内容。 这意味着如果文档在拍摄快照的时间和处理索引请求之间发生更改，则会出现版本冲突。 当版本匹配时，文档会更新，版本号会递增。

> **备注：由于内部版本控制不支持将值0作为有效版本号，因此无法使用\_update\_by\_query更新版本等于零的文档，并且将使请求失败。**

所有更新和查询失败都会导致`_update_by_query`中止，并在响应失败时返回。 已执行的更新仍然存在。 换句话说，该过程不会回滚，只会中止。 当第一个故障导致中止时，失败的批量请求返回的所有故障都将在failure元素中返回; 因此，可能存在相当多的失败实体。

如果您只想简单地计算版本冲突，不要导致`_update_by_query`中止，您可以设置在URL中设置 `conflicts=proceed` 或在请求体中设置`"conflicts": "proceed"`。 第一个例子是这样做的，因为它只是试图获取在线映射更改，而版本冲突只是意味着在`_update_by_query`开始和尝试更新文档之间更新了冲突文档。 这很好，因为该更新将获得在线映射更新。

回到API格式，这将更新来自`twitter`索引的推文：

```
POST twitter/_doc/_update_by_query?conflicts=proceed
```

您还可以使用查询DSL限制`_update_by_query`。 这将更新用户`kimchy`的`twitter`索引中的所有文档：

```
POST twitter/_update_by_query?conflicts=proceed
{
  "query": {                   // 1
    "term": {
      "user": "kimchy"
    }
  }
}

1.必须以与Search API相同的方式将查询作为值传递给query键。 您也可以使用与搜索API相同的q参数。
```

到目前为止，我们只是在不更改文档`source`的情况下更新文档。 这对于[picking up new properties](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#picking-up-a-new-property)非常有用，但这只是其中一半的乐趣。 `_update_by_query`支持脚本来更新文档。 这将增加所有`kimchy`的推文上的`likes`字段：

```
POST twitter/_update_by_query
{
  "script": {
    "source": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

就像在Update API中一样，您可以设置`ctx.op`来更改执行的操作：

```
noop
    如果您的脚本确定它不需要进行任何更改，请设置ctx.op =“noop”。 这将导致_update_by_query从其更新中省略该文档。 
    这种无操作将在noop计数器中报告。
delete
    如果脚本决定必须删除文档，请设置ctx.op =“delete”。 删除将在响应正文中的deleted计数器中报告。
```

将`ctx.op`设置为其他任何内容都是错误的。 在`ctx`中设置任何其他字段是错误的。

请注意，我们没有指定`conflict = proceed`。 在这种情况下，我们希望版本冲突中止该过程，以便我们可以处理失败。

此API不允许您移动它接触的文档，只需修改它们的`source`。 这是故意的！ 我们没有规定将文档从原始位置删除。

也可以同时在多个索引和多个类型上完成这一切，就像搜索API一样：

```
POST twitter,blog/_doc,post/_update_by_query
```

如果提供路由，则路由将复制到滚动查询，将进程限制为与该路由值匹配的分片：

```
POST twitter/_update_by_query?routing=1
```

默认情况下`_update_by_query`使用1000的滚动批次。您可以使用`scroll_size `URL参数更改批量大小：

```
POST twitter/_update_by_query?scroll_size=100
```

\_update\_by\_query也可以通过指定这样的`pipeline`来使用“[摄取节点](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html)”功能：

```
PUT _ingest/pipeline/set-foo
{
  "description" : "sets foo",
  "processors" : [ {
      "set" : {
        "field": "foo",
        "value": "bar"
      }
  } ]
}
POST twitter/_update_by_query?pipeline=set-foo
```

### URL参数

除了像`pretty`之类的标准参数之外，Update By Query API还支持`refresh，wait_for_completion，wait_for_active_shards，timeout`和`scroll`。

发送`refresh`将在更新请求完成时，更新参与索引更新的所有分片。 这与Index API的refresh参数不同，后者仅导致接收新数据的分片被编入索引。

如果请求包含`wait_for_completion = false`，则Elasticsearch将执行一些预检检查，启动请求，然后返回可与Tasks API一起使用的`task`，以取消或获取任务的状态。 Elasticsearch还将在`.tasks / task / $ {taskId}`中创建此任务的记录作为文档。 这是你的保留或删除你认为合适。 完成后，删除它，以便Elasticsearch可以回收它使用的空间。

`wait_for_active_shards`控制在继续请求之前必须激活碎片的副本数。 详情[请见此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards)。 timeout指示每个写入请求等待不可用分片变为可用的时间。 两者都完全适用于Bulk API中的工作方式。 由于\_update\_by\_query使用滚动搜索，您还可以指定scroll参数来控制“搜索上下文”保持活动的时间，例如`？scroll = 10m`，默认情况下为5分钟。

`requests_per_second`可以设置为任何正十进制数（1.4,6,1000等）和限制速率，其中`_update_by_query`通过用等待时间填充每个批来发出批量索引操作。 可以通过将`requests_per_second`设置为-1来禁用限制。

通过在批处理之间等待来完成限制，以便`_update_by_query`在内部使用的滚动可以被赋予考虑填充的超时。 填充时间是批量大小除以`requests_per_second`和写入时间之间的差值。 默认情况下，批处理大小为1000，因此如果requests\_per\_second设置为500：

```
target_time = 1000 / 500 per second = 2 seconds
wait_time = target_time - write_time = 2 seconds - .5 seconds = 1.5 seconds
```

由于批处理是作为单个`_bulk`请求发出的，因此批量大小将导致Elasticsearch创建许多请求，然后等待一段时间再开始下一个集合。 这是“突发的”而不是“平滑de”。 默认值为-1。

### 返回体

JSON返回结果：

```
{
  "took" : 147,
  "timed_out": false,
  "total": 5,
  "updated": 5,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "failures" : [ ]
}
```

```
took
    整个操作从开始到结束的毫秒数。
timed_out
    如果在查询执行更新期间执行的任何请求超时，则此标志设置为true。
total
    已成功处理的文档数。
updated
    已成功更新的文档数。
deleted
    已成功删除的文档数。
batches
    由查询更新拉回的滚动响应数。
version_conflicts
    按查询更新的版本冲突数。
noops
    由于用于查询更新的脚本而忽略的文档数返回了ctx.op的noop值。
retries
    逐个更新尝试的重试次数。 bulk是重试的批量操作数，search是重试的搜索操作数。
throttled_millis
    请求睡眠以符合requests_per_second的毫秒数。
requests_per_second
    在查询更新期间有效执行的每秒请求数。
throttled_until_millis
    在_update_by_query响应中，此字段应始终等于零。 它只在使用Task API时有意义，它指示下一次（自纪元以来的毫秒），
    将再次执行受限制的请求以符合requests_per_second。
failures
    如果在此过程中存在任何不可恢复的错误，则会出现故障数组。 如果这是非空的，那么请求因为那些失败而中止。逐个查询是使用批处理实现的，
    任何故障都会导致整个进程中止，但当前批处理中的所有故障都会被收集到数组中。 您可以使用conflict选项来防止reindex在版本冲突中中止。
```

### 实用TASK API

您可以使用[Task API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)获取所有正在运行的update-by-query请求的状态：

```
GET _tasks?detailed=true&actions=*byquery
```

结果：

```
{
  "nodes" : {
    "r1A2WoRbTwKZ516z6NEs5A" : {
      "name" : "r1A2WoR",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1:9300",
      "attributes" : {
        "testattr" : "test",
        "portsfile" : "true"
      },
      "tasks" : {
        "r1A2WoRbTwKZ516z6NEs5A:36619" : {
          "node" : "r1A2WoRbTwKZ516z6NEs5A",
          "id" : 36619,
          "type" : "transport",
          "action" : "indices:data/write/update/byquery",
          "status" : {                                             // 1
            "total" : 6154,
            "updated" : 3500,
            "created" : 0,
            "deleted" : 0,
            "batches" : 4,
            "version_conflicts" : 0,
            "noops" : 0,
            "retries": {
              "bulk": 0,
              "search": 0
            },
            "throttled_millis": 0
          },
          "description" : ""
        }
      }
    }
  }
}

1.该对象包含实际状态。 它就像响应json一样，是对整体字段的重要补充。 total是reindex期望执行的操作总数。
 您可以通过添加updated，created和deleted的字段来估计进度。 当它们的总和等于total字段时，请求将结束。
```

使用任务ID，您可以直接查找任务。 以下示例检索有关任务r1A2WoRbTwKZ516z6NEs5A：36619的信息：

此API的优点是它与`wait_for_completion = false`集成，以透明地返回已完成任务的状态。 如果任务完成并且在其上设置了`wait_for_completion = false`，则它将返回结果或错误字段。 此功能的成本是`wait_for_completion = false`在.`tasks / task / $ {taskId}`创建的文档。 您可以删除该文档。

### 实用取消Task API

可以使用任务取消API取消任何Update By Query：

```
POST _tasks/r1A2WoRbTwKZ516z6NEs5A:36619/_cancel
```

取消应该很快发生，但可能需要几秒钟。 上面的任务状态API将继续列出任务，直到它被唤醒以取消自身。

### 限制

可以使用\_rethrottle API 修改正在运行update by query 的requests\_per\_second的值：

```
POST _update_by_query/r1A2WoRbTwKZ516z6NEs5A:36619/_rethrottle?requests_per_second=-1
```

就像在\_update\_by\_query API上设置它时，requests\_per\_second可以是-1来禁用限制，也可以是1.7或12之类的任何十进制数来限制到该级别。 加速查询的Rethrottling会立即生效，但是降低查询速度，会在完成当前批处理后生效， 这可以防止滚动超时

### 切片

逐个查询支持切片滚动以并行化更新过程。 这种并行化可以提高效率，并提供一种方便的方法将请求分解为更小的部分。

### 手动分片

通过为每个请求提供切片ID和切片总数，手动切片查询：

```
POST twitter/_update_by_query
{
  "slice": {
    "id": 0,
    "max": 2
  },
  "script": {
    "source": "ctx._source['extra'] = 'test'"
  }
}
POST twitter/_update_by_query
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "script": {
    "source": "ctx._source['extra'] = 'test'"
  }
}
```

可以验证：

```
GET _refresh
POST twitter/_search?size=0&q=extra:test&filter_path=hits.total
```

一个合理的`total`

```
{
  "hits": {
    "total": 120
  }
}
```

### 自动切片

您还可以使用“[切片滚动](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll)”自动并行查询以在\_uid上切片。 使用切片指定要使用的切片数：

```
POST twitter/_update_by_query?refresh&slices=5
{
  "script": {
    "source": "ctx._source['extra'] = 'test'"
  }
}
```

验证：

```
POST twitter/_search?size=0&q=extra:test&filter_path=hits.total
```

合理的total数：

```
{
  "hits": {
    "total": 120
  }
}
```

将`slices`设置为`auto`将允许Elasticsearch选择要使用的切片数。 此设置将使用每个分片一个切片，达到一定限制。 如果有多个源索引，它将根据具有最小分片数的索引选择切片数。

向\_update\_by\_query添加切片只会自动执行上一节中使用的手动过程，创建子请求，这意味着它有一些怪癖：



