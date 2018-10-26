# 删除角色映射API

删除角色映射

### 请求

```
DELETE /_xpack/security/role_mapping/<name>
```

### 描述

角色映射定义了每个用户分配了那些角色。更多信息见：[Mapping users and groups to roles](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/mapping-roles.html)

#### 路径参数

```
name
    (string)定义角色映射的不同名字。这个名字仅仅只是促进API内部交互。在任何情况下都不会影响映射行为。
```

### 授权

集群的`manage_security`权限（或者更高权限）

### 样例

下例删除用户角色映射：

```
DELETE /_xpack/security/role_mapping/mapping1
```

如果成功删除角色，请求返回 `{"found": true}` 。否则 `found`返回 false。

```
{
  "found" : true
}
```



