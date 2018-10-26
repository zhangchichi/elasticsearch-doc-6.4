# 删除用户API

在本地域中删除用户

### 请求

```
DELETE /_xpack/security/user/<username>
```

### 路径参数

```
username
    (string)用户标识
```

### 授权

集群的`manage_security`权限（或者更高权限）

### 样例

删除用户 `jacknich`

```
DELETE /_xpack/security/user/jacknich
```

如果成功删除角色，请求返回 `{"found": true}` 。否则 `found`返回 false。

```
{
  "found" : true
}
```



