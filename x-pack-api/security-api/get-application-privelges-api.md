# 获取应用程序权限API

返回 [application privileges](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/security-privileges.html#application-privileges)。

### 请求

`GET /_xpack/security/privilege`

`GET /_xpack/security/privilege/<application>`

`GET /_xpack/security/privilege/<application>/<privilege>`

### 描述

要检查用户的应用程序权限，请使用has privileges API。

### 路径参数

```
application
    （string）应用程序的名称。 应用程序权限始终与一个应用程序相关联。 如果未指定此参数，API将返回所有关所有应用程序的所有权限的信息。
privilege
    （string）权限的名称。 如果未指定此参数，API将返回有关所请求应用程序的所有权限的信息。
```

### 授权

要使用此API,必须要具有以下一项：

* 集群的`manage_security`权限（或者更高权限）
* 请求引用了应用程序的_"Manage Application Privileges"全局权限_

### 样例

以下示例检索有关`app01`应用程序的`read`权限的信息：

```
GET /_xpack/security/privilege/myapp/read
```

成功调用将返回由应用程序名称和权限名称键入的对象。 如果未定义权限，则请求将以404状态响应。

```
{
  "myapp": {
    "read": {
      "application": "myapp",
      "name": "read",
      "actions": [
        "data:read/*",
        "action:login"
      ],
      "metadata": {
        "description": "Read access to myapp"
      }
    }
  }
}
```

要检索应用程序的所有权限，请省略权限名称：

```
GET /_xpack/security/privilege/myapp/
```

要检索每个权限，请省略应用程序和权限名称：

```
GET /_xpack/security/privilege/
```



