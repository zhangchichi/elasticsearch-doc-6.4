# Term Vectors

返回特定文档字段中词项（term）的信息和统计信息。 该文档可以存储在索引中或由用户人工提供。 词项向量默认是实时的，而不是接近实时的。 可以通过将`realtime`参数设置为false来更改此设置。

```
GET /twitter/_doc/1/_termvectors
```

（可选）您可以使用url中的参数指定要为其检索信息的字段

```
GET /twitter/_doc/1/_termvectors?fields=message
```

或者在请求正文中添加请求的字段（参见下面的示例）。 还可以使用通配符以与[多匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)类似的方式指定字段

**警告：**请注意，/\_termvector的使用在2.0中已弃用，并替换为/\_termvectors。

### 返回值

可以请求三种类型的值：词项信息（_term information_），词项统计（_term statistics_）和字段统计（_field statistics_）。 默认情况下，为所有字段返回所有词项信息（_term information_）和字段统计（_field statistics_），但不返回词项信息（_term information_）。

#### 词项信息（_term information_）

* 字段中词项频率（一直返回）
* 词项位置\(`positions`: true\)
* 开始和结束偏移\(`offsets`: true\)
* 词项负载（payloads：true），作为base64编码的字节

如果请求的信息未存储在索引中，则将尽可能快速计算。 另外，可以针对甚至不存在于索引中的文档计算词项向量，而是由用户提供。

**注意：**开始和结束偏移假定使用UTF-16编码。 如果要使用这些偏移量来获取生成此标记的原始文本，则应确保使用UTF-16编码的字符串也是使用UTF-16进行编码的。

#### 词项统计（_term statistics_）

设置term\_statistics为true\(默认为false\)将会返回：

* 总词项频率（词项在所有文档中出现的频率）
* 文件频率（包含当前词项的文件数）

默认情况下，不会返回这些值，因为词项统计信息会对性能产生严重影响。

#### 字段统计（_field statistics_）

将`field_statistics`设置为`false`（默认为true）将省略：

* 文档计数（包含此字段的文档数）
* 文件频率之和（该字段中所有词项的文件频率之和）
* 总词项频率之和（该字段中每个词项的总词项频率之和）

#### 词项过滤

使用参数`filter`，还可以根据其tf-idf分数过滤返回词项。 这对于找出文档的良好特征向量可能是有用的。 此功能的工作方式与“更喜欢此查询”的第二阶段类似（[second phase](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html#mlt-query-term-selection) of the [More Like This Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html)）。 有关用法，请参见示例5。

自持下列子参数：

| `max_num_terms` | 每个字段必须返回的最大词项数。 默认为25。 |
| :--- | :--- |
| `min_term_freq` | 忽略源文档中频率低于此频率的单词。 默认为1。 |
| `max_term_freq` | 忽略源文档中超过此频率的单词。 默认为无限制。 |
| `min_doc_freq` | 忽略至少在这么多文档中没有出现的术语。 默认为1。 |
| `max_doc_freq` | 忽略超过这么多文档中出现的单词。 默认为无限制。 |
| `min_word_length` | 最小字长，低于该字长将被忽略。 默认为0。 |
| `max_word_length` | 最大字长，高于该字长将被忽略。 默认为无界（0）。 |

### 表现

词项和字段统计数据不准确。删除的文档不会被考虑在内。 仅为请求的文档所在的分片检索信息。因此，术语和字段统计仅用作相对度量，而绝对数量在此上下文中没有意义。 默认情况下，在请求人工文档的术语向量时，随机选择用于获取统计数据的分片。 仅使用路由来命中特定的分片。



