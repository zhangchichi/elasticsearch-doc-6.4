# 创建或者修改应用程序权限API

新增或者修改[应用程序权限](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/security-privileges.html#application-privileges)

#### 请求

```
POST /_xpack/security/privilege
```

```
PUT /_xpack/security/privilege
```

#### 描述

此API创建或者修改权限。删除权限，请使用[删除应用程序权限API](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-delete-privilege.html)

更多信息，详见[应用程序权限](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/defining-roles.html#roles-application-priv)

检查用户的应用程序权限，使用[是否有权限API](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-has-privileges.html)

#### 请求体

请求体时一个JSON对象，其中字段名称是应用程序名称，字段值是一个对象。内部对象的字段名称是权限，字段值是一个包含下面字段的JSON对象：

```
actions
    (array-of-string)此权限授予的操作名称列表，此字段必须存在，并且不能为空数组。
metadata
    (object)选填元数据。metadata对象内部以_开头的关键字，是系统保留的
```

#### 验证

_**应用程序名称**_

应用程序名称由前缀组成，可选后缀符合下列规则：

* 前缀必须以ASCII字母开头
* 前缀只能包含ASCII字母和数组
* 前缀长度必须大于等于3
* 后缀必须以-或者\_开头
* 后缀不能包含以下字符：`\\, /, *, ?, ", <, >, |, ,, *`
* 名称不能包含空格

_**权限名称**_

权限名称必须以小写ACSII开头，并且只能包含ASCII字母和数字已经\_,-和. 

_**操作名称**_

操作名称能够包含任何能打印的ASCII字符，并且必须包含下列字符中的一个` / * , :`

#### 授权

要使用此API,必须要具有以下一项：

* 集群的`manage_security`权限（或者更高权限）
* 请求引用了应用程序的_"Manage Application Privileges"全局权限_

#### 样例

提交一个PUT或者POST请求`/_xpack/security/privilege/<application>/<privilege>` 到节点，来新增一个单独的权限。例如：

```
PUT /_xpack/security/privilege
{
  "myapp": {
    "read": {
      "actions": [       // 1
        "data:read/*" ,  // 2
        "action:login" ],
        "metadata": {    // 3
          "description": "Read access to myapp"
        }
      }
    }
}
```

1.actions字符串数组对“myapp”有重要意思。elasticsearch不对他们赋予含义。

2.这里的通配符 （\*），意味着所有以"data:read/\*" 开头的操作都被授予了权限。elasticsearch对这些操作不赋予任何意思。但是如果请求包含应用应用程序权限请求，例如`data:read/users`  或者 `data:read/settings，`  ， [has privileges API](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-has-privileges.html) 会使用通配符，返回true.

3.metadata对象是可选的

无论是新增或者修改成功，会返回一个JSON结构

```
{
  "myapp": {
    "read": {
      "created": true   //1
    }
  }
}
```

1.在更新已存在的权限时，created 返回false

要新增多个权限，提交POSTq请求`/_xpack/security/privilege/`到服务端，例如：

```
PUT /_xpack/security/privilege
{
  "app01": {
    "read": {
      "actions": [ "action:login", "data:read/*" ]
    },
    "write": {
      "actions": [ "action:login", "data:write/*" ]
    }
  },
  "app02": {
    "all": {
      "actions": [ "*" ]
    }
  }
}
```

无论是新增或者修改成功，会返回一个JSON结构

```
{
  "app02": {
    "all": {
      "created": true
    }
  },
  "app01": {
    "read": {
      "created": true
    },
    "write": {
      "created": true
    }
  }
}
```



