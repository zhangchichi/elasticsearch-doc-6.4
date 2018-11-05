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

默认情况下，\_delete\_by\_query使用1000的滚动批次。您可以使用`scroll_size `URL参数更改批量大小：

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



