# SQL REST API

SQL REST API接受JSON文档中的SQL，执行它并返回结果。 例如：

```
POST /_xpack/sql?format=txt
{
    "query": "SELECT * FROM library ORDER BY page_count DESC LIMIT 5"
}
```

结果：

```
     author      |        name        |  page_count   | release_date
-----------------+--------------------+---------------+------------------------
Peter F. Hamilton|Pandora's Star      |768            |2004-03-02T00:00:00.000Z
Vernor Vinge     |A Fire Upon the Deep|613            |1992-06-01T00:00:00.000Z
Frank Herbert    |Dune                |604            |1965-06-01T00:00:00.000Z
Alastair Reynolds|Revelation Space    |585            |2000-03-15T00:00:00.000Z
James S.A. Corey |Leviathan Wakes     |561            |2011-06-02T00:00:00.000Z
```

虽然text / plain格式对人类来说很好，但计算机更喜欢更有结构的东西。 你可以用以下格式替换格式的值：

```
-json 又名 application/json 
-yaml 又名 application/yaml 
-smile 又名 application/smile 
-cbor 又名 application/cbor 
-txt 又名 text/plain 
-csv 又名 text/csv 
-tsv 又名 text/tab-separated-values
```



