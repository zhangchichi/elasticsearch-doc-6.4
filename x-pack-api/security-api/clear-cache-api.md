# 清除缓存API

此API将把用户从用户缓存中清除。你可以全部清除或清除指定用户

### 请求

```
POST _xpack/security/realm/<realms>/_clear_cache
```

```
POST _xpack/security/realm/<realms>/_clear_cache?usernames=<usernames>
```

### 描述

用户凭证缓存在每个节点的缓存中，这样可以避免连接到远程认证服务或者每一次请求查询硬盘。你可以通过领域设置来配置用户缓存。更多信息见 [Controlling the User Cache](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/controlling-user-cache.html)。

要把角色从角色缓存中清除，见 [Clear Roles Cache API](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-clear-role-cache.html)。

### 路径参数

```
realms
    (list)要清除的以逗号分隔的列表
usernames
    (list)要从内存中清除的以逗号分割的用户列表。如果不指定该参数，此API清除用户内存中的所有用户
```

### 样例

清除所有缓存在 file 域中的用户，例如：

```
POST _xpack/security/realm/default_file/_clear_cache
```

指定 usernames 参数，清除所有的指定用户，例如：

```
POST _xpack/security/realm/default_file/_clear_cache?usernames=rdeniro,alpacino
```

清除多个域的缓存，以逗号分割指定域列表，例如：

```
POST _xpack/security/realm/default_file,ldap1/_clear_cache
```



