# 禁用用户API

在本地域中禁用用户

### 请求

```
PUT /_xpack/security/user/<username>/_disable
```

### 描述

默认情况下，创建用户时会启用用户。 可以使用此API撤消用户对Elasticsearch的访问权限。 要重新启用用户，需要[启用用户API](//x-pack-api/security-api/enable-user-api.md)。

### 路径参数

```
username(必填)
    (string)用户标识
```

### 授权

集群的`manage_security`权限（或者更高权限）

### 样例

```
PUT /_xpack/security/user/jacknich/_disable
```



