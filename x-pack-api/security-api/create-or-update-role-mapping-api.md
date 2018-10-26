# 创建或修改角色映射API

创建或修改角色映射

#### 请求

```
POST /_xpack/security/role_mapping/<name>
```

```
PUT /_xpack/security/role_mapping/<name>
```

#### 描述

角色映射定义每个用户被分配了那些角色。每个映射包含用户定义规则和授予这些用户的角色列表。

> **备注：此api不创建新的角色。相反，它将用户映射到现有角色。创建角色可以使用 **[**角色管理apis **](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-roles.html)**和 **[**角色文件**](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/defining-roles.html#roles-management-file)。

更多信息查看[映射用户和组到角色](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/mapping-roles.html)

#### 路径参数

```
name
    (string)定义角色映射的唯一名称。该名称仅作为标识符，以方便API交互。它不会以任何方式影响映射的行为。
```

#### 请求体

一下参数可以被指定到PUT和POST的请求体中，他们与新增一个角色映射有关：

```
enabled(必须)
    (boolean)执行角色映射时，将会忽略enable设置为false的映射。
metadata
    (object)有助于定义每个用户被分配了那些角色的其他元数据，在metadata对象中,以_开头的关键字是为系统使用保留的。
roles(必须)
    (list)被授予用户的角色列表，与角色映射规则相匹配，
rules(必须)
    (object)确定映射匹配那些用户的规则。每个规则是使用JSON DSL表示的逻辑条件。    
```

[角色映射资源](https://www.elastic.co/guide/en/elasticsearch/reference/current/role-mapping-resources.html)

#### 授权

使用此API,至少需要拥有集群 `manage_security `权限

#### 样例

以下例子把user角色分配给了所有用户：

```
POST /_xpack/security/role_mapping/mapping1
{
  "roles": [ "user"],
  "enabled": true,  //执行角色映射时，将会忽略enable设置为false的映射
  "rules": {
    "field" : { "username" : "*" }
  },
  "metadata" : {    //可选项
    "version" : 1
  }
}
```

无论时创建还是修改映射，成功将返回一个JSON结构

```
{
  "role_mapping" : {
    "created" : true   //在更新映射时会返回false
  }
}
```

以下例子将uesr和admin角色分配给指定用户：

```
POST /_xpack/security/role_mapping/mapping2
{
  "roles": [ "user", "admin" ],
  "enabled": true,
  "rules": {
     "field" : { "username" : [ "esadmin01", "esadmin02" ] }
  }
}
```

下面的例子匹配用户名为`esadmin`或者在`cn=admin,dc=example,dc=com`组中的用户：

```
POST /_xpack/security/role_mapping/mapping3
{
  "roles": [ "superuser" ],
  "enabled": true,
  "rules": {
    "any": [
      {
        "field": {
          "username": "esadmin"
        }
      },
      {
        "field": {
          "groups": "cn=admins,dc=example,dc=com"
        }
      }
    ]
  }
}
```

下面的例子针对特定领域认证的用户：

```
POST /_xpack/security/role_mapping/mapping4
{
  "roles": [ "ldap-user" ],
  "enabled": true,
  "rules": {
    "field" : { "realm.name" : "ldap1" }
  }
}
```

下面的例子匹配在特定LDAP子树中的用户：

```
POST /_xpack/security/role_mapping/mapping5
{
  "roles": [ "example-user" ],
  "enabled": true,
  "rules": {
    "field" : { "dn" : "*,ou=subtree,dc=example,dc=com" }
  }
}
```

下面的例子匹配在指定域中的特定LDAP子树的用户：

```
POST /_xpack/security/role_mapping/mapping6
{
  "roles": [ "ldap-example-user" ],
  "enabled": true,
  "rules": {
    "all": [
      { "field" : { "dn" : "*,ou=subtree,dc=example,dc=com" } },
      { "field" : { "realm.name" : "ldap1" } }
    ]
  }
}
```

规则可以跟复制，并且包含通配符匹配。例如：下面的映射匹配满足所有条件的任何用户：

* 名称匹配`*,ou=admin,dc=example,dc=com`模式，名称为`es-admin` ，或者名称为`es-system`
* 用户属于`cn=people,dc=example,dc=com`组
* 用户不含有`terminated_date`

```
POST /_xpack/security/role_mapping/mapping7
{
  "roles": [ "superuser" ],
  "enabled": true,
  "rules": {
    "all": [
      {
        "any": [
          {
            "field": {
              "dn": "*,ou=admin,dc=example,dc=com"
            }
          },
          {
            "field": {
              "username": [ "es-admin", "es-system" ]
            }
          }
        ]
      },
      {
        "field": {
          "groups": "cn=people,dc=example,dc=com"
        }
      },
      {
        "except": {
          "field": {
            "metadata.terminated_date": null
          }
        }
      }
    ]
  }
}
```



