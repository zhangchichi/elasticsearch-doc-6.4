# 安全性

如果在群集上启用了安全性，则Elasticsearch SQL会整合到其中。 在这种情况下，Elasticsearch SQL支持传输层的安全性（通过加密使用者和服务器之间的通信）和身份验证（对于访问层）。

### SSL/TLS 配置

如果是加密传输，则需要在Elasticsearch SQL中启用SSL / TLS支持，以正确建立与Elasticsearch的通信。 这可以通过将`ssl`属性设置为`true`或使用URL中的`https`前缀来完成。根据您的SSL配置（证书是否由CA签名，无论它们是JVM级别的全局证书还是仅一个应用程序的本地证书），可能需要设`keystore`和/或`truststore`，即存储凭据的位置 （`keystore` - 通常存储私钥和证书）以及如何验证它们（`truststore` - 通常存储来自第三方的证书，也称为CA - 证书颁发机构）。通常（并且再次注意您的环境可能会有很大差异），如果Elasticsearch SQL的SSL设置尚未在JVM级别完成，则需要设置`keystore`，如果Elasticsearch SQL安全性需要客户端身份验证（PKI - 公钥 基础设施），并在启用SSL时设置`truststore`。

### 认证

在Elasticsearch中支持两种认证方式：

**Username/Password**

通过用户和密码属性设置

**PKI/X.509**

使用X.509证书对Elasticsearch SQL进行Elasticsearch身份验证。为此，需要将包含私钥和证书的密钥库设置为适当的用户（在Elasticsearch中配置）和具有CA证书的信任库，用于在Elasticsearch集群中签署SSL / TLS证书。也就是说，应该设置密钥来验证Elasticsearch SQL，并验证它是否正确。 为此，应该设置ssl.keystore.location和ssl.truststore.location属性以指示要使用的密钥库和信任库。 建议通过密码保护这些安全，在这种情况下需要ssl.keystore.pass和ssl.truststore.pass属性。

### 权限（服务端）

最后，一个服务器需要向用户添加一些权限，以便他们可以运行SQL。 要运行SQL，用户需要 `read `和 `indices：admin/get `权限至少，而API的某些部分需要 `cluster：monitor / main`。

```
cli_or_jdbc_minimal:
  cluster:
    - "cluster:monitor/main"
  indices:
    - names: test
      privileges: [read, "indices:admin/get"]
    - names: bort
      privileges: [read, "indices:admin/get"]
```



