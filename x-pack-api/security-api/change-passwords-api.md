# 修改密码API

在本地域中修改用户密码

### 请求

```
POST _xpack/security/user/_password
```

```
POST _xpack/security/user/<username>/_password
```

### 描述

你可以使用 [create user api](/x-pack-api/security-api/create-or-update-user-api.md) 去更新除了 `username` 和 `password` 的任何用户属性。此API修改用户密码。

关于本地域的更多信息，见 [Realms](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/realms.html) 和 [Configuring a native realm](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-native-realm.html) 。

### 路径参数

```
username
    (string) 你想修改密码的用户，如果不指定用户名那么修改当前用户密码
```

### 请求体

```
password(必填)
    
    (string)新的用户密码
```

### 授权

任何用户可以修改自己的密码。拥有`manage_security` 权限的用户可以修改其他用户密码。

### 样例

下面的例子修改用户`jacknich` 的密码：

```
POST /_xpack/security/user/jacknich/_password
{
  "password" : "s3cr3t"
}
```

成功返回一个JSON结构

```
{}
```



