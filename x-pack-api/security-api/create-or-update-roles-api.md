# 创建和修改角色API

在本地域（`native realm`）中新增和修改角色

#### 请求

```
POST /_xpack/security/role/<name>
```

```
PUT /_xpack/security/role/<name>
```

#### 描述

角色API通常是管理角色的首选方式，而不是使用基于文件的角色管理方式。本地域的更多信息，查看[Realms](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/realms.html) 和 [Configuring a native realm](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-native-realm.html)

#### 路径参数

```
name
    (string)角色名称
```

#### 请求体

可以在PUT和POST请求的主体中指定以下参数，这些参数与添加角色有关：

```
applications
     (list)应用程序权限条目列表
     application（必须）
          (string)此条目适用的应用名称
     privileges
          (list)字符串列表，其中每个元素都是应用程序权限和操作的名称
     resources
          (list)应用权限的资源列表
cluster
     (list)集群权限列表，这些权限定义此角色能执行的集群级别的操作
global
     (object)定义全局权限的对象，全局权限是一种请求感知的集群特权形式。目前对全局权限的支持仅限于管理应用程序权限。这是一个可选向。
indices
     (list)索引权限条目列表
     field_security
          (list)角色所有者具有读取权限的文档字段。更多信息见 设置文档和字段级别安全。
     names(必须)
          (list)此条目中的权限适用的索引列表（或者是索引名称适配模式）
     privileges(必须)
          (list)角色所有者在指定索引上拥有的索引级别权限
     query
          搜索查询，定义角色所有者对文档的读取权限。指定索引的文档必须与查询匹配，这样角色拥有者才能读取。     
metadata
     (object)可选元数据，元数据对象中，以_开头的关键字会被系统留用。
run_as
     (list)此角色所有者可以模拟的用户列表。更多信息，查看 以其他用户行为提交请求。
```

更多信息，查看 [角色定义。](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/defining-roles.html)

#### 授权

要使用此API，至少需要拥有 `manage_security`集群权限

#### 样例

以下示例添加一个名为 `my_admin_role `的角色：

```
POST /_xpack/security/role/my_admin_role
{
  "cluster": ["all"],
  "indices": [
    {
      "names": [ "index1", "index2" ],
      "privileges": ["all"],
      "field_security" : { // optional
        "grant" : [ "title", "body" ]
      },
      "query": "{\"match\": {\"title\": \"foo\"}}" // optional
    }
  ],
  "applications": [
    {
      "application": "myapp",
      "privileges": [ "admin", "read" ],
      "resources": [ "*" ]
    }
  ],
  "run_as": [ "other_user" ], // optional
  "metadata" : { // optional
    "version" : 1
  }
}
```

成功返回一个JSON结构，该结构展示角色是否被创建或更新成功：

```
{
  "role": {
    "created": true  //在更新一个已存在的角色的时候，created为false
  }
}
```



