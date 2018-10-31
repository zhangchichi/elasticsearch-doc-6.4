# 数据类型

Elasticsearch SQL中使用了大部分的Elasticsearch[数据类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)，如下所示：

| Elasticsearch type | SQL type | SQL precision |
| :---: | :---: | :---: |


| Core types |
| :---: |


| null | null | 0 |
| :--- | :--- | :--- |
| boolean | boolean  | 1 |
| byte | tinyint | 3 |
| short | smallint | 5 |
| integer | integer | 10 |
| long | long | 19 |
| double | double | 15 |
| float | real | 7 |
| halt\_float | float | 16 |
| scaled\_float | float | 19 |
| keyword | varchar | 基于 ignore\_above |
| text | varchar | 2,147,483,647 |
| binary | varbinary | 2,147,483,647 |
| date | timestamp | 24 |

| 复杂类型 |
| :---: |


| object | struct | 0 |
| :--- | :--- | :--- |
| nested | struct | 0 |

| 不支持得类型 |
| :---: |


| 上面没提到得类型 | 不支持 | 0 |
| :--- | :--- | :--- |


显然，并非Elasticsearch中的所有类型都具有SQL中的等价物，反之亦然，因此，Elasticsearch SQL使用前者的数据类型特性而不是后者，因为最终Elasticsearch是后备存储。

### SQL 和 多领域

Elasticsearch的一个核心概念是分析字段，它是为了有效地索引而解释的全文值。这些字段是文本类型，不用于排序或聚合，因为它们的实际值取决于所使用的分析器，因此Elasticsearch还提供了用于存储精确值的关键字类型。在大多数情况下，实际上是默认的，当Elasticsearch通过多个字段支持的字符串时，使用这两种类型，即能够以多种方式索引相同的字符串; 例如，将其作为搜索文本索引，也作为排序和聚合的关键字。

> 这里需要 查看 mapping之后再回过来看

由于SQL需要精确值，因此在遇到文本字段时，Elasticsearch SQL将搜索可用于比较，排序和聚合的精确多字段。 为此，它将搜索它可以找到的未标准化的第一个关键字，并将其用作原始字段的精确值。

思考下面的 `string `映射：

```
{
    "first_name" : {
        "type" : "text",
        "fields" : {
            "raw" : {
                "type" : "keyword"
            }
        }
    }
}
```

下面的sql 查询：

```
SELECT first_name FROM index WHERE first_name = 'John'
```

等同于：

```
SELECT first_name FROM index WHERE first_name.raw = 'John'
```

因为Elasticsearch SQL会自动从`raw`中获取`raw`多字段以进行精确匹配。













































