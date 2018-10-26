# 删除角色API

删除本地域中的角色

### 请求

```
DELETE /_xpack/security/role/<name>
```

### 描述

相较于本地文件管理角色，角色API通常是管理角色的首选。本地于的更多信息见：[Realms](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/realms.html) and [Configuring a native realm](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-native-realm.html)。

### 路径参数

```
name
    (string)角色名称
```

### 授权

集群的`manage_security`权限（或者更高权限）

### 样例

下例删除 `my_admin_role`  角色：

```
DELETE /_xpack/security/role/my_admin_role
```

如果成功删除角色，请求返回 `{"found": true}` 。否则 `found`返回 false。

```
{
  "found" : true
}
```



