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
     (object)
```



