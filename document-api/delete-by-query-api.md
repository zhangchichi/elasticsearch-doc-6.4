# Delete by Query API

最简单的`_delete_by_query`用法只会对匹配查询的每个文档执行删除操作。 这是API：

```
POST twitter/_delete_by_query
{
  "query": {      //1
    "match": {
      "message": "some message"
    }
  }
}

1. 必须以与Search API相同的方式将查询作为值传递给查询键。 您也可以使用与搜索API相同的q参数。
```

返回结果大致如下：

```
{
  "took" : 147,
  "timed_out": false,
  "deleted": 119,
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
  "total": 119,
  "failures" : [ ]
}
```

`_delete_by_query`在启动时获取索引的快照，并使用内部版本控制删除它找到的内容。 这意味着如果文档在拍摄快照的时间和处理删除请求之间发生更改，则会出现版本冲突。 当版本匹配时，文档将被删除。

> 备注：由于内部版本控制不支持将值0作为有效版本号，因此无法使用\_delete\_by\_query删除版本等于零的文档，并且将使请求失败。

在`_delete_by_query`执行期间，顺序执行多个搜索请求以查找要删除的所有匹配文档。每次找到一批文档时，都会执行相应的批量请求以删除所有这些文档。 如果搜索或批量请求被拒绝，`_delete_by_query`依赖于默认策略来重试被拒绝的请求（最多10次，指数退回）。 达到最大重试次数限制会导致`_delete_by_query`中止，并且在响应失败时返回所有失败。 已执行的删除仍然有效。 换句话说，该过程不会回滚，只会中止。 当第一个故障导致中止时，失败的批量请求返回的所有故障都将在failure元素中返回; 因此，可能存在相当多的失败实体。

如果你想计算版本冲突而不是让它们中止，那么在url中设置`conflicts=proceed`或在请求正文中设置`"conflicts": "proceed"`。

```
POST twitter/_doc/_delete_by_query?conflicts=proceed
{
  "query": {
    "match_all": {}
  }
}
```

想一次删除多个indexs或者多个type,只需要像下面的SEARCH api一样：

```
POST twitter,blog/_docs,post/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

如果提供路由，则路由将复制到滚动查询，将进程限制为与该路由值匹配的分片：

```
POST twitter/_delete_by_query?routing=1
{
  "query": {
    "range" : {
        "age" : {
           "gte" : 10
        }
    }
  }
}
```

默认情况下，\_delete\_by\_query使用1000的滚动批次。您可以使用`scroll_size`URL参数更改批量大小：

```
POST twitter/_delete_by_query?scroll_size=5000
{
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

### URL参数

除了像`pretty`这样的标准参数之外，Delete By Query API还支持`refresh`，`wait_for_completion`，`wait_for_active_shards`，`timeout`和`scroll`。

发送`refresh`将刷新请求完成后按查询删除所涉及的所有分片。 这与Delete API的refresh参数不同，后者只会刷新收到删除请求的分片。

如果请求包含`wait_for_completion = false`，则Elasticsearch将执行一些预检检查，启动请求，然后返回可与Tasks API一起使用的任务，以取消或获取任务的状态。 Elasticsearch还将在`.tasks / task / $ {taskId}`中创建此任务的记录作为文档。 这是你的保留或删除你认为合适。 完成后，删除它，以便Elasticsearch可以回收它使用的空间。

`wait_for_active_shards`控制在继续请求之前必须激活碎片的副本数。 详情请见此处。 `timeout`指示每个写入请求等待不可用分片变为可用的时间。 两者都完全适用于Bulk API中的工作方式。 如果`_delete_by_query`使用滚动搜索，您还可以指定scroll参数来控制“搜索上下文”保持活动的时间，例如？`scroll = 10m`，默认情况下为5分钟。

`requests_per_second`可以设置为任何正十进制数（1.4,6,1000等）和限制速率，在此速率下，`_delete_by_query`通过用等待时间填充每个批次来发出批量删除操作。 可以通过将`requests_per_second`设置为-1来禁用限制。

通过在批处理之间等待来完成限制，以便\_delete\_by\_query在内部使用的滚动可以被赋予考虑填充的超时。 填充时间是批量大小除以requests\_per\_second和写入时间之间的差异。 默认情况下，批处理大小为1000，因此如果requests\_per\_second设置为500：

```
target_time = 1000 / 500 per second = 2 seconds
wait_time = target_time - write_time = 2 seconds - .5 seconds = 1.5 seconds
```

由于批处理是作为单个\_bulk请求发出的，因此批量数太大将导致Elasticsearch创建许多请求，然后等待一段时间再开始下一个集合。 这是“突发”而不是“平滑”。 默认值为-1。

### 返回主体（response body）

```
{
  "took" : 147,
  "timed_out": false,
  "total": 119,
  "deleted": 119,
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
    操作从开始到结束花费的毫秒数
time_out
    如果在查询执行删除期间执行的任何请求超时，则此标志设置为true。
total
    成功处理的文档数量
deleted
    成功删除的文档数量
batches
    通过查询删除拉回的滚动响应数。
version_conflicts
    按查询删除的版本冲突数。
noops
    对于按查询删除，此字段始终等于零。 它只存在，以便通过查询删除，按查询更新和reindex API返回具有相同结构的响应。
retries
    通过查询删除尝试的重试次数。 bulk是重试的批量操作数，search是重试的搜索操作数。
throttled_millis
    请求睡眠以符合requests_per_second的毫秒数。
requests_per_second
    在通过查询删除期间有效执行的每秒请求数。
throttled_until_millis
    在_delete_by_query响应中，此字段应始终等于零。 它只在使用Task API时有意义，它指示下一次（自纪元以来的毫秒），将再次执行受限制的请求以符合requests_per_second。
failures
    如果在此过程中存在任何不可恢复的错误，则会出现故障数组。 如果这是非空的，那么请求因为那些失败而中止。 逐个查询是使用批处理实现的，任何故障都会导致整个进程中止，但当前批处理中的所有故障都会被收集到数组中。 您可以使用conflict选项来防止reindex在版本冲突中中止。
```

### 使用TASK API

您可以使用Task API获取任何正在运行的delete-by-query 请求的状态：

```
GET _tasks?detailed=true&actions=*/delete/byquery
```

回复：

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
          "action" : "indices:data/write/delete/byquery",
          "status" : {                                                  //1
            "total" : 6154,
            "updated" : 0,
            "created" : 0,
            "deleted" : 3500,
            "batches" : 36,
            "version_conflicts" : 0,
            "noops" : 0,
            "retries": 0,
            "throttled_millis": 0
          },
          "description" : ""
        }
      }
    }
  }
}

1.该对象包含实际状态。 它就像响应json一样，是全局的重要补充。 total是reindex期望执行的操作总数。 您可以通过
  updated, created, deleted字段来估计进度。 当它们的总和等于总字段时，请求将结束。
```

使用taskid可以直接查看：

```
GET /_tasks/r1A2WoRbTwKZ516z6NEs5A:36619
```

此API的优点是它与`wait_for_completion = false`集成，以透明地返回已完成任务的状态。 如果任务完成并且在其上设置了`wait_for_completion = false`，那么它将返回结果或错误字段。 此功能的成本是wait\_for\_completion = false在`.tasks / task / $ {taskId}`创建的文档。 您可以删除该文档。

### 使用取消 TASK API

所有 delete by query API 可以使用它取消任务：

```
POST _tasks/r1A2WoRbTwKZ516z6NEs5A:36619/_cancel
```

可以使用任务API找到任务ID。

取消应该很快发生，但可能需要几秒钟。 上面的任务状态API将继续列出任务，直到它被唤醒以取消自身。

### 更新限制

可以使用\_rethrottle API通过查询在正在运行的删除时更改`requests_per_second`的值：

```
POST _delete_by_query/r1A2WoRbTwKZ516z6NEs5A:36619/_rethrottle?requests_per_second=-1
```

就像在\_delete\_by\_query API上设置它时，requests\_per\_second可以是-1来禁用限制或任何十进制数字如1.7或12来限制到该级别。 加速查询的Rethrottling会立即生效，但是在完成当前批处理后，重新启动会降低查询速度。 这可以防止滚动超时。

### 切片

逐个查询支持切片滚动以并行化删除过程。 这种并行化可以提高效率，并提供一种方便的方法将请求分解为更小的部分。

### 手动切片

通过为每个请求提供切片ID和切片总数，手动切片查询：

```
POST twitter/_delete_by_query
{
  "slice": {
    "id": 0,
    "max": 2
  },
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
POST twitter/_delete_by_query
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
```

您可以验证哪些适用：

```
GET _refresh
POST twitter/_search?size=0&filter_path=hits.total
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
```

这导致像这样一个合理的总数：

```
{
  "hits": {
    "total": 0
  }
}
```

### 自动分片：

您还可以使用“切片滚动”自动并行查询，以便在\_uid上进行切片。 使用`slices`指定要使用的切片数：

```
POST twitter/_delete_by_query?refresh&slices=5
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
```

您还可以验证以下内容：

```
POST twitter/_search?size=0&filter_path=hits.total
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
```

这导致像这样一个合理的`total`：

```
{
  "hits": {
    "total": 0
  }
}
```

将切片设置为auto将允许Elasticsearch选择要使用的切片数。 此设置将使用每个分片一个切片，达到一定限制。 如果有多个源索引，它将根据具有最小分片数的索引选择切片数。

向\_delete\_by\_query添加切片只会自动执行上一节中使用的手动过程，创建子请求，这意味着它有一些怪癖：

* 您可以在Tasks API中查看这些请求。 这些子请求是具有切片的请求的任务的“子”任务。
* 使用切片获取请求的任务状态仅包含已完成切片的状态。
* 这些子请求可单独寻址，例如取消和重新限制。
* 使用`slice`限制请求将按比例重新调整未完成的子请求。
* 使用`slice`取消请求将取消每个子请求。
* 由于`slice`的性质，每个子请求都不会获得完全均匀的文档分布。 所有文档被处理，但某些切片可能比其他分片更大。 期望更大的切片具有更均匀的分布。
* 像slice\_per\_second这样的参数和带有切片的请求的大小按比例分配给每个子请求。 结合上面关于分布不均匀的点，您应该得出结论，使用切片的大小可能不会导致确切大小的文档为\`\_delete\_by\_query\`ed。
* 每个子请求获得的源索引的略有不同的快照，尽管这些都是在大约相同的时间进行的。

### 选择分片数量

如果自动切片，将切片设置为自动将为大多数索引选择合理的数字。 如果您手动切片或以其他方式调整自动切片，请使用这些指南。

当切片数等于索引中的分片数时，查询性能最有效。 如果该数字很大（例如，500），则选择较小的数字，因为太多的切片会损害性能。 设置高于分片数量的片通常不会提高效率并增加开销。

删除性能在可用资源上以切片数量线性扩展。

查询或删除性能是否主导运行时取决于重新编制索引的文档和群集资源。



