# Reindex API

Reindex要求为源索引中的所有文档启用\_source。

Reindex不会尝试设置目标索引。 它不会复制源索引的设置。 您应该在运行\_reindex操作之前设置目标索引，包括设置映射，分片计数，副本等。

\_reindex的最基本形式只是将文档从一个索引复制到另一个索引。 这会将twitter索引中的文档复制到new\_twitter索引中：

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

返回像这样的结果：

```
{
  "took" : 147,
  "timed_out": false,
  "created": 120,
  "updated": 0,
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

就像`_update_by_query`一样，\_reindex获取源索引的快照，但其目标必须是不同的索引，因此不太可能发生版本冲突。 dest元素可以像索引API一样配置，以控制乐观并发控制。 只是省略version\_type（如上所述）或将其设置为internal将导致Elasticsearch盲目地将文档转储到目标中，覆盖任何碰巧具有相同类型和id的文件：

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "internal"
  }
}
```

将`version_type`设置为`external`将导致Elasticsearch保留源中的版本，创建缺少的任何文档，并更新目标索引中具有旧版本的文档而不是更新源索引中的所有文档：

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  }
}
```

设置op\_type to create将导致\_reindex仅在目标索引中创建缺少的文档。 所有现有文档都会导致版本冲突：

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  }
}
```

默认情况下，版本冲突会中止\_reindex进程，但您可以通过设置`"conflicts": "proceed"`在请求正文中：

```
POST _reindex
{
  "conflicts": "proceed",
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  }
}
```

您可以通过向源添加类型或添加查询来限制文档。 这只会将`kimchy`的`twitter`复制到new\_twitter中：

```
POST _reindex
{
  "source": {
    "index": "twitter",
    "type": "_doc",
    "query": {
      "term": {
        "user": "kimchy"
      }
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

`source`中的`index`和`type`都可以是列表，允许您在一个请求中从许多源复制。 这将从`twitter`和`blog`索引中复制`_doc`和`post`类型的文档。

```
POST _reindex
{
  "source": {
    "index": ["twitter", "blog"],
    "type": ["_doc", "post"]
  },
  "dest": {
    "index": "all_together",
    "type": "_doc"
  }
}
```

> 注意：Reindex API不会处理ID冲突，因此最后编写的文档将“获胜”，但顺序通常不可预测，因此依赖此行为并不是一个好主意。 相反，请确保使用script使ID唯一。

也可以通过设置`size`来限制处理文档的数量。 这只会将一个文档从twitter复制到new\_twitter：

```
POST _reindex
{
  "size": 1,
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

如果你想要来自twitter索引的特定文档集，你需要使用sort。 排序使滚动效率降低，但在某些情况下，它是值得的。 如果可能，请选择更具选择性的查询来进行大小和排序。 这会将10000个文件从twitter复制到new\_twitter：

```
POST _reindex
{
  "size": 10000,
  "source": {
    "index": "twitter",
    "sort": { "date": "desc" }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

`source`部分支持[搜索请求](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)中支持的所有元素。 例如，可以使用`_source`过滤原始文档中的一部分字段，如下所示：

```
POST _reindex
{
  "source": {
    "index": "twitter",
    "_source": ["user", "_doc"]
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

与`_update_by_query`一样，`_reindex`支持修改文档的script。 与`_update_by_query`不同，允许脚本修改文档的元数据。 这个例子颠覆了source文档的版本：

```
POST _reindex
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  },
  "script": {
    "source": "if (ctx._source.foo == 'bar') {ctx._version++; ctx._source.remove('foo')}",
    "lang": "painless"
  }
}
```

就像在`_update_by_query`中一样，您可以设置`ctx.op`来更改在目标索引上执行的操作：

```
noop
    如果脚本确定文档不必在目标索引中编入索引，请设置ctx.op =“noop”。 这种无操作将在响应机构的noop计数器中报告。
delete
    如果脚本确定必须从目标索引中删除文档，请设置ctx.op =“delete”。 删除将在响应正文中的已删除计数器中报告。
```

将ctx.op设置为其他任何内容都将返回错误，就像在ctx中设置任何其他字段。

想想可能性！ 小心点; 你可以改变：

* `_id`
* `_type`
* `_index`
* `_version`
* `_routing`

将\_version设置为null或从ctx映射中清除它就像不在索引请求中发送版本一样; 无论目标版本或您在\_reindex请求中使用的版本类型如何，它都会导致文档被覆盖在目标索引中。

默认情况下，如果\_reindex看到带有路由的文档，则除非脚本更改了路由，否则将保留路由。 您可以在dest请求上设置路由以更改此设置：

```
keep
    将针对每个匹配发送的批量请求的路由设置为匹配上的路由。 这是默认值。
discard
    将针对每个匹配发送的批量请求的路由设置为null。
=<some text>
    将针对每个匹配发送的批量请求的路由设置为=之后的所有文本。
```

例如，您可以使用以下请求将源索引中包含公司名称cat的所有文档复制到dest索引，并将路由设置为cat。

```
POST _reindex
{
  "source": {
    "index": "source",
    "query": {
      "match": {
        "company": "cat"
      }
    }
  },
  "dest": {
    "index": "dest",
    "routing": "=cat"
  }
}
```

默认情况下，\_reindex使用1000的滚动批次。您可以使用source元素中的size字段更改批量大小：

```
POST _reindex
{
  "source": {
    "index": "source",
    "size": 100
  },
  "dest": {
    "index": "dest",
    "routing": "=cat"
  }
}
```

Reindex还可以通过指定如下管道来使用“[摄取节点](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html)”功能：

```
POST _reindex
{
  "source": {
    "index": "source"
  },
  "dest": {
    "index": "dest",
    "pipeline": "some_ingest_pipeline"
  }
}
```

### 远程reindex

Reindex支持从远程Elasticsearch集群重建索引：

```
POST _reindex
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200",
      "username": "user",
      "password": "pass"
    },
    "index": "source",
    "query": {
      "match": {
        "test": "data"
      }
    }
  },
  "dest": {
    "index": "dest"
  }
}
```

`host`参数必须包含方案，主机，端口（例如https：// otherhost：9200）和可选路径（例如https：// otherhost：9200 / proxy）。 `username`和`password`参数是可选的，当它们存在时`_reindex`将使用基本身份验证连接到远程Elasticsearch节点。 使用基本身份验证时务必使用https，否则密码将以纯文本格式发送。

必须使用`reindex.remote.whitelist`属性在elasticsearch.yaml中将远程主机明确列入白名单。 它可以设置为允许的远程主机和端口组合的逗号分隔列表（例如otherhost:9200，another:9200，127.0.10.\*:9200，localhost:\*）。 白名单忽略Scheme - 仅使用主机和端口，例如：

```
reindex.remote.whitelist: "otherhost:9200, another:9200, 127.0.10.*:9200, localhost:*"
```

必须在将协调reindex的任何节点上配置白名单。

此功能适用于您可能会找到的任何Elasticsearch版本的远程群集。 这应该允许您通过从旧版本的群集重新索引从任何版本的Elasticsearch升级到当前版本。

要启用发送到旧版Elasticsearch的查询，请将查询参数直接发送到远程主机，而不进行验证或修改。

**备注：**从远程群集重新索引不支持[手动](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-manual-slice)或[自动切片](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-automatic-slice)。

从远程服务器重新索引使用堆上缓冲区，默认最大大小为100mb。 如果远程索引包含非常大的文档，则需要使用较小的批处理大小。 下面的示例将批量大小设置为10，非常非常小。

```
POST _reindex
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200"
    },
    "index": "source",
    "size": 10,
    "query": {
      "match": {
        "test": "data"
      }
    }
  },
  "dest": {
    "index": "dest"
  }
}
```

也可以使用`socket_timeout`字段在远程连接上设置套接字读取超时，并使用`connect_timeout`字段设置连接超时。 两者都默认为30秒。 此示例将套接字读取超时设置为一分钟，将连接超时设置为10秒：

```
POST _reindex
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200",
      "socket_timeout": "1m",
      "connect_timeout": "10s"
    },
    "index": "source",
    "query": {
      "match": {
        "test": "data"
      }
    }
  },
  "dest": {
    "index": "dest"
  }
}
```

### URL参数

参见[此处](//document-api/update-by-query-api.md)

### 返回主体

JSON结构的返回：

```
{
  "took": 639,
  "timed_out": false,
  "total": 5,
  "updated": 0,
  "created": 5,
  "deleted": 0,
  "batches": 1,
  "noops": 0,
  "version_conflicts": 2,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": 1,
  "throttled_until_millis": 0,
  "failures": [ ]
}
```

```
took
    整个操作所花费的总毫秒数。
timed_out
    如果在reindex期间执行的任何请求超时，则此标志设置为true。
total
    已成功处理的文档数。
updated
    已成功更新的文档数。
created
    已成功创建的文档数。
deleted
    已成功删除的文档数。
batches
    重新索引拉回的滚动响应数。
noops
    由于用于reindex的脚本返回了ctx.op的noop值而被忽略的文档数。
version_conflicts
    重新索引的版本冲突数。
retries
    reindex尝试重试的次数。 bulk是重试的批量操作数，search是重试的搜索操作数。
throttled_millis
    请求睡眠以符合requests_per_second的毫秒数。
requests_per_second
    在reindex期间有效执行的每秒请求数。
throttled_until_millis
    在_reindex响应中，此字段应始终等于零。它只在使用Task API时有意义，它指示下一次（自纪元以来的毫秒），将再次执行受限制的请求以符合requests_per_second。
failures
    如果在此过程中存在任何不可恢复的错误，则会出现故障数组。如果这是非空的，那么请求因为那些失败而中止。
    Reindex是使用批处理实现的，任何故障都会导致整个进程中止，但当前批处理中的所有故障都会被收集到数组中。
    您可以使用conflict选项来阻止reindex在版本冲突时中止。使用connect_timeout字段超时。两者都默认为30秒。
    此示例将套接字读取超时设置为一分钟，将连接超时设置为10秒：
```

### 使用TASK API

参见[此处](//document-api/update-by-query-api.md)

### 使用取消TASK API

参见[此处](//document-api/update-by-query-api.md)

### 限制

参见[此处](//document-api/update-by-query-api.md)

### ReIndex改变字段名称

\_reindex可用于构建具有重命名字段的索引副本。 假设您创建一个包含如下所示文档的索引：

```
POST test/_doc/1?refresh
{
  "text": "words words",
  "flag": "foo"
}
```

但你不喜欢名称flag，并希望用tab替换它。 \_reindex可以为您创建另一个索引：

```
POST _reindex
{
  "source": {
    "index": "test"
  },
  "dest": {
    "index": "test2"
  },
  "script": {
    "source": "ctx._source.tag = ctx._source.remove(\"flag\")"
  }
}
```

查询新文档：

```
GET test2/_doc/1
```

返回：

```
{
  "found": true,
  "_id": "1",
  "_index": "test2",
  "_type": "_doc",
  "_version": 1,
  "_source": {
    "text": "words words",
    "tag": "foo"
  }
}
```

### 分片

Reindex支持[Sliced Scroll](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll)以并行化重建索引过程。 这种并行化可以提高效率，并提供一种方便的方法将请求分解为更小的部分。

#### 手动分片

通过为每个请求提供切片ID和切片总数，手动切片reindex请求：

```
POST _reindex
{
  "source": {
    "index": "twitter",
    "slice": {
      "id": 0,
      "max": 2
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
POST _reindex
{
  "source": {
    "index": "twitter",
    "slice": {
      "id": 1,
      "max": 2
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

验证：

```
GET _refresh
POST new_twitter/_search?size=0&filter_path=hits.total
```

合理的total返回：

```
{
  "hits": {
    "total": 120
  }
}
```

### 自动分片

您还可以让\_reindex使用切片滚动自动并行化以在\_uid上切片。 使用切片指定要使用的切片数：

```
POST _reindex?slices=5&refresh
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

验证：

```
POST new_twitter/_search?size=0&filter_path=hits.total
```

合理的total返回：

```
{
  "hits": {
    "total": 120
  }
}
```

详情参见[此处](//document-api/update-by-query-api.md)

### Reindex多索引

如果你有很多索引要重新索引，通常最好一次重新索引它们，而不是使用glob模式来获取许多索引。 这样，如果有任何错误，您可以通过删除部分完成的索引并从该索引重新开始来恢复该过程。 它还使得流程的并行化非常简单：将索引列表拆分为reindex并并行运行每个列表。

一个似乎很适合的bash脚本：

```
for index in i1 i2 i3 i4 i5; do
  curl -HContent-Type:application/json -XPOST localhost:9200/_reindex?pretty -d'{
    "source": {
      "index": "'$index'"
    },
    "dest": {
      "index": "'$index'-reindexed"
    }
  }'
done
```

### Reindex日常索引

尽管有上述建议，您可以将`_reindex`与[Painless](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html)结合使用以重新索引每日索引，以将新模板应用于现有文档。

假设您有包含以下文档的索引：

```
PUT metricbeat-2016.05.30/_doc/1?refresh
{"system.cpu.idle.pct": 0.908}
PUT metricbeat-2016.05.31/_doc/1?refresh
{"system.cpu.idle.pct": 0.105}
```

metricbement- \*索引的新模板已加载到Elasticsearch中，但它仅适用于新创建的索引。[Painless](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html)可用于重新索引现有文档并应用新模板。

下面的脚本从索引名称中提取日期，并创建一个附加了-1的新索引。 来自metricbeat-2016.05.31的所有数据将重新编入metricbeat-2016.05.31-1。

```
POST _reindex
{
  "source": {
    "index": "metricbeat-*"
  },
  "dest": {
    "index": "metricbeat"
  },
  "script": {
    "lang": "painless",
    "source": "ctx._index = 'metricbeat-' + (ctx._index.substring('metricbeat-'.length(), ctx._index.length())) + '-1'"
  }
}
```

现在可以在\* -1索引中找到以前的metricbeat索引中的所有文档。

```
GET metricbeat-2016.05.30-1/_doc/1
GET metricbeat-2016.05.31-1/_doc/1
```

之前的方法也可以与更改字段名称一起使用，以仅将现有数据加载到新索引中，并在需要时重命名任何字段。

### 随机提取索引子集

\_reindex可用于提取索引的随机子集以进行测试：

```
POST _reindex
{
  "size": 10,
  "source": {
    "index": "twitter",
    "query": {
      "function_score" : {
        "query" : { "match_all": {} },
        "random_score" : {}
      }
    },
    "sort": "_score"          // 1
  },
  "dest": {
    "index": "random_twitter"
  }
}

1. _reindex默认按_doc排序，因此除非您将排序重写为_score，否则random_score不会产生任何影响。
```



