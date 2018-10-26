# 清除角色缓存API

在本地角色缓存中清除角色

### 请求

```
POST /_xpack/security/role/<name>/_clear_cache
```

### 描述

本地域的详情见 [Realms](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/realms.html) and [Configuring a native realm](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-native-realm.html)。

### 路径参数

```
name
    (string)角色名称
```

### 授权

要使用此API，你必须至少拥有集群 `manage_security`权限

### 样例

清除本地角色缓存，例如，清除 `my_admin_role `的缓存：

```
POST /_xpack/security/role/my_admin_role/_clear_cache
```



