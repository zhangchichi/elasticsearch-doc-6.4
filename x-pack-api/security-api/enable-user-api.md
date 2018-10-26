# 启用用户API

在本地域中启用用户

### 请求

```
PUT /_xpack/security/user/<username>/_enable
```

### 描述

默认情况下，创建用户时会启用它们。 您可以使用此`enable users API`和`diable user api`来更改该属性。

有关本机领域的更多信息，请参阅领域和配置本机领域。

### 路径参数

```
username(必填)
    (string)用户标识
```

### 授权

集群的`manage_security`权限（或者更高权限）

### 样例

```
PUT /_xpack/security/user/jacknich/_enable
```



