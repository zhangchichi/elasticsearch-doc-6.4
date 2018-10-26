# 删除令牌API

废除一个无需基础认证的请求的承载令牌。

> 原文：Invalidates a bearer token for access without requiring basic authentication.
>
> 意思是，一个请求没有使用基础认证，而使用token来访问。此api可以让这个token失效

### 请求

```
DELETE /_xpack/security/oauth2/token
```

### 描述

[获取令牌AP](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-get-token.html)I 返回一个具有有效时间的令牌，在这段时间之后令牌将再也无法使用。该时间由`xpack.security.authc.token.timeout` 设置。更多信息见 [Token service settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#token-service-settings) [edit](https://github.com/elastic/elasticsearch/edit/6.4/docs/reference/settings/security-settings.asciidoc) 。

使用此API,可以立刻使令牌无效。

### 请求体

```
token
    (string)一个访问令牌
```

### 样例

使指定令牌立刻失效：

```
DELETE /_xpack/security/oauth2/token
{
  "token" : "dGhpcyBpcyBub3QgYSByZWFsIHRva2VuIGJ1dCBpdCBpcyBvbmx5IHRlc3QgZGF0YS4gZG8gbm90IHRyeSB0byByZWFkIHRva2VuIQ=="
}
```

返回一个JSON结构

```
{
  "created" : true  //如果已失效，created 返回 false
}
```



