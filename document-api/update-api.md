# Update API

更新API允许基于提供的脚本更新文档。 该操作从索引获取文档（与分片并置），运行脚本（使用可选的脚本语言和参数），并对结果进行索引（也允许删除或忽略操作）。 它使用版本控制来确保在“get”和“reindex”期间没有发生更新。

请注意，此操作仍然意味着文档的完全重新索引，它只是删除了一些网络往返，并减少了get和index之间版本冲突的可能性。 需要启用\_source字段才能使此功能正常工作。

例如，让我们索引一个简单的文档：

```
PUT test/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

### 脚本更新

现在，我们可以执行一个增加计数器的脚本：

```
POST test/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
```

我们可以在标签列表中添加一个标签（注意，如果标签存在，它仍会添加它，因为它是一个列表）：

```
POST test/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params" : {
            "tag" : "blue"
        }
    }
}
```

除了\_source 下列参数也存在在 `cxt `map中：`_index`,`_type`,`_id`,`_version`,`_routing`和`_now（当前时间戳）`

也可以在文档中添加新的字段：

```
POST test/_doc/1/_update
{
    "script" : "ctx._source.new_field = 'value_of_new_field'"
}
```

或者删除一个字段：

```
POST test/_doc/1/_update
{
    "script" : "ctx._source.remove('new_field')"
}
```

而且，我们甚至可以改变执行的操作。 如果`tags`字段包含`green`，此示例将删除`doc`，否则它不执行任何操作（`noop`）：

```
POST test/_doc/1/_update
{
    "script" : {
        "source": "if (ctx._source.tags.contains(params.tag)) { ctx.op = 'delete' } else { ctx.op = 'none' }",
        "lang": "painless",
        "params" : {
            "tag" : "green"
        }
    }
}
```

### 使用部分文档进行更新

更新API还支持传递部分文档，该部分文档将合并到现有文档中（简单的递归合并，对象的内部合并，替换核心“键/值”和数组）。 要完全替换现有文档，应使用`index API`。 以下部分更新会向现有文档添加新字段：

```
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    }
}
```

如果同时指定了doc和script，则会忽略doc。 最好是将部分文档的字段对放在脚本本身中。

### 检测noop更新

如果指定了doc，则其值将与现有\_source合并。 默认情况下，不更改任何内容的更新会检测到它们没有更改任何内容并返回`“result”：“noop”`，如下所示：

```
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    }
}
```

如果在发送请求之前name是new\_name，则忽略整个更新请求。 如果忽略请求，响应中的result元素将返回noop。

```
{
   "_shards": {
        "total": 0,
        "successful": 0,
        "failed": 0
   },
   "_index": "test",
   "_type": "_doc",
   "_id": "1",
   "_version": 6,
   "result": "noop"
}
```

您可以通过设置“detect\_noop”来禁用此行为：false，如下所示：

```
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    },
    "detect_noop": false
}
```

### Upserts

如果文档尚不存在，则upsert元素的内容将作为新文档插入。 如果文档确实存在，那么将执行脚本：

```
POST test/_doc/1/_update
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}
```

**scripted\_upsert**

如果您希望脚本运行，无论文档是否存在 - 即脚本处理初始化文档而不是`upsert`元素 - 然后将`scripted_upsert`设置为true：

```
POST sessions/session/dh3sgudg8gsrgl/_update
{
    "scripted_upsert":true,
    "script" : {
        "id": "my_web_session_summariser",
        "params" : {
            "pageViewEvent" : {
                "url":"foo.com/bar",
                "response":404,
                "time":"2014-01-01 12:32"
            }
        }
    },
    "upsert" : {}
}
```

**doc\_as\_upsert**

将doc\_as\_upsert设置为true将使用doc的内容作为upsert值，而不是发送部分doc加上upsert文档：

```
POST test/_doc/1/_update
{
    "doc" : {
        "name" : "new_name"
    },
    "doc_as_upsert" : true
}
```

### 参数

更新操作支持以下查询字符串参数：

| `retry_on_conflict` | 在更新的get和indexing阶段之间，另一个进程可能已经更新了同一文档。 默认情况下，更新将因版本冲突异常而失败。 retry\_on\_conflict参数控制在最终抛出异常之前重试更新的次数。 |
| :--- | :--- |
| routing | 路由用于将更新请求路由到正确的分片，并在更新的文档不存在时为upsert请求设置路由。 不能用于更新现有文档的路由。 |
| timeout | 超时等待碎片变为可用。 |
| wait\_for\_active\_shards | 在继续更新操作之前需要处于活动状态的分片副本数。 详情请见此处。 |
| refresh | 控制何时此请求所做的更改对搜索可见。 看？refresh。 |
| \_source | 允许控制是否以及如何在响应中返回更新的源。 默认情况下，不会返回更新的源。 请参阅源过滤了解详细信息 |
| version | 更新API在内部使用Elasticsearch的版本控制支持，以确保在更新期间文档不会更改。 您可以使用version参数指定仅在文档版本与指定版本匹配时才更新文档。 |

更新API不支持内部版本以外的版本控制

更新API不支持外部（版本类型external＆external\_gte）或强制（版本类型强制）版本控制，因为它会导致Elasticsearch版本号与外部系统不同步。 请改用index API。

