# 删除应用程序权限API

删除[应用程序权限](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/security-privileges.html#application-privileges)

### 请求

```
DELETE /_xpack/security/privilege/<application>/<privilege>
```

### 路径参数

```
application（必填）
    (string)应用程序权限。应用程序权限始终与一个应用程序相关联。
privilege(必填)
    (string)权限名称
```

### 授权

要使用此API,必须要具有以下一项：

* 集群的`manage_security`权限（或者更高权限）
* 请求引用了应用程序的_"Manage Application Privileges"全局权限_

### 样例

下面样例，删除了应用程序 `myapp `的 `read `应用程序权限：

```
DELETE /_xpack/security/privilege/myapp/read
```

如果成功删除角色，请求放回 `{"found": true}` 。否则 `found `返回 false。

```
{
  "myapp": {
    "read": {
      "found" : true
    }
  }
}
```



