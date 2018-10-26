```
request:
PUT /test_index_1 HTTP/1.1
Host: localhost:9200
Authorization: Basic ZWxhc3RpYzpjaGFuY2UxMjM=
Cache-Control: no-cache
Postman-Token: fa52803e-9164-ccff-f734-981d8a82b74f
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

response:
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "test_index_1"
}

```



写入document

```
PUT /test_index_1/_doc/1 HTTP/1.1
Host: localhost:9200
Content-Type: application/json
Authorization: Basic Y2M6Y2hhbmNlMTIz
Cache-Control: no-cache
Postman-Token: c376d1f3-3423-d25e-0e21-5b4b97145819

{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}


赋权前：
response:
{
    "error": {
        "root_cause": [
            {
                "type": "security_exception",
                "reason": "action [indices:data/write/index] is unauthorized for user [cc]"
            }
        ],
        "type": "security_exception",
        "reason": "action [indices:data/write/index] is unauthorized for user [cc]"
    },
    "status": 403
}



赋权后：
response:
{
    "_index": "test_index_1",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 2
}
```



