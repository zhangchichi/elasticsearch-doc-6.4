建立一个能够读取，写入 test\_index\_1 的角色

```
request:
PUT /_xpack/security/role/read_test_index_1 HTTP/1.1
Host: localhost:9200
Content-Type: application/json
Authorization: Basic ZWxhc3RpYzpjaGFuY2UxMjM=
Cache-Control: no-cache
Postman-Token: 6c971717-aa46-011d-a523-f4a4da5a2414

{
  "indices": [
    {
      "names": [ "test_index_1"],
      "privileges": ["write"]
    }
  ]
}

response:
{
    "role": {
        "created": true
    }
}
```



