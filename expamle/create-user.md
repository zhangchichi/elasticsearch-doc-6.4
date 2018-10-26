创建用户

```
request:
PUT /_xpack/security/user/cc HTTP/1.1
Host: localhost:9200
Content-Type: application/json
Authorization: Basic ZWxhc3RpYzpjaGFuY2UxMjM=
Cache-Control: no-cache
Postman-Token: 69078eaf-3a31-7cd9-acbd-972b14bc3497

{
  "password" : "chance123",
  "roles" : [ "read_test_index_1" ],
  "full_name" : "zhang cc",
  "email" : "cc@example.com"
}

response:
{
    "user": {
        "created": true
    }
}
```

关联角色：



