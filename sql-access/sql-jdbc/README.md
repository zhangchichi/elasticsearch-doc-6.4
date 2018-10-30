# SQL JDBC

Elasticsearch的SQL jdbc驱动程序是Elasticsearch的丰富，功能齐全的JDBC驱动程序。 它是Type 4驱动程序，这意味着它是：a platform independent, stand-alone, Direct to Database, pure Java，它将JDBC调用转换为Elasticsearch SQL。

### 安装

JDBC驱动程序可以通过从elastic.co站点下载或使用具有以下依赖关系的Maven兼容工具获得：

```
<dependency>
  <groupId>org.elasticsearch.plugin</groupId>
  <artifactId>x-pack-sql-jdbc</artifactId>
  <version>6.4.2</version>
</dependency>
```

添加`artifacts.elastic.co/maven`仓库：

```
<repositories>
  <repository>
    <id>elastic.co</id>
    <url>https://artifacts.elastic.co/maven</url>
  </repository>
</repositories>
```

### 建立

驱动程序主类是org.elasticsearch.xpack.sql.jdbc.jdbc.JdbcDriver。 请注意，驱动程序实现JDBC 4.0服务提供程序机制，这意味着只要在类路径中可用，它就会自动注册。

注册后，驱动程序将以下语法理解为URL：

```
jdbc:es://[http|https]?[host[:port]]*/[prefix]*[?[option=value]&]*
jdbc:es:// 前缀。强制。
[http|https] 要进行的HTTP连接类型 - http（默认）或https。 可选的。
[host[:port]] host（默认为localhost）和port（默认为9200）。 可选的。
[prefix] 前缀（默认为空）。 通常在特定路径下托管Elasticsearch时使用。 可选的。
[?[option=value]&] JDBC驱动程序的参数。 默认为空。 可选的。
```

驱动识别下列参数：

**必要**

```
timezone (默认为 JVM timezone)
    每个连接使用的时区由其ID指示。 强烈建议将其设置为（比如说，UTC），因为JVM时区可能会有所不同，
    对于整个JVM来说是全局的，并且在安全管理器下运行时无法轻松更改。
```

**网络**

```
connect.timeout (默认30s)
    连接超时（以秒为单位）。 这是等待与服务器建立连接的最长时间。
network.timeout (默认60s)
    网络超时（以秒为单位）。 这是等待网络的最长时间。
page.timeout (默认45s)
    页面超时（以秒为单位）。 这是等待页面的最长时间。
page.size (默认1000)
    页面大小（在条目中）。 服务器每页返回的结果数。
query.timeout (默认90s)
    查询超时（以秒为单位）。 这是等待查询返回的最长时间。
```

**基础认证**

```
user
    用户名
password
    密码
```

**SSL**

```
ssl (默认false)
    开启SSL
ssl.keystore.location
    key store (如果开启) 存储地址
ssl.keystore.pass
    key store 密码
ssl.keystore.type (默认JKS)
    key store 类型. PKCS12是一种常见的替代格式
ssl.truststore.location
    trust store 地址
ssl.truststore.pass
    trust store 密码
ssl.cert.allow.self.signed (默认false)
    是否允许自签名证书
ssl.protocol(default TLS)
    使用的SSL协议
```

**协议**

```
proxy.http
    Http代理主机名
proxy.socks
    socks代理主机名
```

下面的URL,放入了所有参数：

```
jdbc:es://http://server:3456/timezone=UTC&page.size=250
```

在端口3456上打开与服务器的Elasticsearch SQL连接，将JDBC连接时区设置为UTC，将其pagesize设置为250个条目。

