# 创建和删除用户API

新增和更新本机域（`native realm`）的用户，这些用户通常被称为本地用户（`native users`）

#### 请求

`POST /_xpack/security/user/<username>`

`PUT /_xpack/security/user/<username>`

#### 描述

在更新用户的过程中，除了`username `和 `password `以外都能修改。要更改用户密码，使用[change password API](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-change-password.html)。

有关本机领域（`native realm`）更多的信息，查看[Realms](https://www.elastic.co/guide/en/elastic-stack-overview/6.4/realms.html) 和 [Configuring a native realm](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-native-realm.html)。

#### 路径参数

`username `\(必须\)  ：（String）用户标识

**note:**

username长度必须在\[1-1024\]的闭合空间内。可以包含字母\(`a-z`,`A-Z`,`0-9`\)，[Basic Latin \(ASCII\) block](https://en.wikipedia.org/wiki/Basic_Latin_%28Unicode_block%29) 列表中的标点和可打印符号。不允许使用前导和尾随空格。

#### 请求体

可以在POST和PUT的请求体中指定下面的参数：

```
enabled
        (boolean) 指定是否启用用户，默认是true
```

```
email
        (string)用户email
```

```
full_name
        (string)用户全名
```

```
metadata
        （object）要与用户关联的任何元数据
```

```
password（必须）
        （string）用户密码，密码长度必须至少6个字符。
```

```
roles(必须)
        （list）用户拥有的一组角色。角色确定了用户的访问权限。创建一个没角色的用户，请指定一个空数组:[]
```

#### 授权

要使用此API,必须至少拥有集群的`manage_security`权限

##### 样例

一下实例创建用户`jacknich`：

```
POST /_xpack/security/user/jacknich
{
  "password" : "j@rV1s",
  "roles" : [ "admin", "other_role1" ],
  "full_name" : "Jack Nicholson",
  "email" : "jacknich@example.com",
  "metadata" : {
    "intelligence" : 7
  }
}
```

成功调用将返回一个JSON结构的结果。显示用户是否创建成功

```
{
  "user": {
    "created" : true   //更新用户成功，created会返回false
  }
}
```

在新增用户之后，可以验证这个用户请求。例如:

```
curl -u jacknich:j@rV1s http://localhost:9200/_cluster/health
```



