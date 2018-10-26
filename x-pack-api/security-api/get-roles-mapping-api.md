# 获取角色映射API

返回角色映射

### 请求

`GET /_xpack/security/role_mapping`

`GET /_xpack/security/role_mapping/<name>`

### 描述

角色映射定义为每个用户分配的角色。 有关更多信息，请参阅将用户和组映射到角色。

### 路径参数

```
name
    （string）标识角色映射的不同名称。 该名称仅用作标识符，以便通过API进行交互; 它不会以任何方式影响映射的行为。
     您可以将多个映射名称指定为逗号分隔列表。 如果未指定此参数，API将返回有关所有角色映射的信息。
```

### 结果

成功调用将检索一个对象，其中键是请求映射的名称，值是这些映射的JSON表示形式。 有关更多信息，请参阅角色映射资源。

如果没有所请求名称的映射，则响应将具有状态代码404。

### 授权

集群的`manage_security`权限（或者更高权限）

### 样例

以下示例检索有关mapping1角色映射的信息：

```
GET /_xpack/security/role_mapping/mapping1
```

```
{
  "mapping1": {
    "enabled": true,
    "roles": [
      "user"
    ],
    "rules": {
      "field": {
        "username": "*"
      }
    },
    "metadata": {}
  }
}
```



