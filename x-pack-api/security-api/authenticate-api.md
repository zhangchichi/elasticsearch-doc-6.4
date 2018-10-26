# 认证API

通过认证API,你可以提交一个头部带基础认证的请求，认证用户并且返回认证通过用户的信息。

### 请求

```
GET _xpack/security/_authenticate
```

### 描述

调用成功返回一个JSON结构，该结构显示分配给用户的角色以及任何已经分配的元数据

如果认证失败，此API返回401状态码

### 样例

要验证用户，提交一个GET请求`_xpack/security/_authenticate` 到终端：

```
GET _xpack/security/_authenticate 
```

下面样例输出提供用户”rdeniro“相关信息：

```
{
  "username": "rdeniro",
  "roles": [
    "admin"
  ],
  "full_name": null,
  "email":  null,
  "metadata": { },
  "enabled": true
}
```



