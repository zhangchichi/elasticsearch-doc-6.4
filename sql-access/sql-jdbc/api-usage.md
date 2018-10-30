# API用法

可以通过官方java.sql和javax.sql包使用JDBC：

### java.sql

通过`java.sql.Driver`和`DriverManager`格式:

```
String address = "jdbc:es://" + elasticsearchAddress;         //1
Properties connectionProperties = connectionProperties();     //2
Connection connection = DriverManager.getConnection(address, connectionProperties);

1.Elasticsearch正在侦听HTTP流量的服务器和端口。 该端口默认为9200。
2.用于连接到Elasticsearch的属性。 对于不安全的Elasticsearch，空的Properties实例很好。
```

### javax.sql

可通过javax.sql.DataSource API访问：

```
JdbcDataSource dataSource = new JdbcDataSource();
String address = "jdbc:es://" + elasticsearchAddress;         //1
dataSource.setUrl(address);
Properties connectionProperties = connectionProperties();     //2
dataSource.setProperties(connectionProperties);
Connection connection = dataSource.getConnection();

1.Elasticsearch正在侦听HTTP流量的服务器和端口。 该端口默认为9200。
2.用于连接到Elasticsearch的属性。 对于不安全的Elasticsearch，空的Properties实例很好。
```

哪一个使用？ 通常，在URL中提供大多数配置参数的客户端应用程序依赖于DriverManager样式，而传递时DataSource是首选，因为它可以在一个地方配置，而消费者只需要调用getConnection而不必担心任何其他参数。

要连接到安全的Elasticsearch服务器，属性应如下所示：

```
Properties properties = new Properties();
properties.put("user", "test_admin");
properties.put("password", "x-pack-test-password");
```

一旦建立连接，就可以像使用任何其他JDBC连接一样使用它。 例如：

```
try (Statement statement = connection.createStatement();
        ResultSet results = statement.executeQuery(
            "SELECT name, page_count FROM library ORDER BY page_count DESC LIMIT 1")) {
    assertTrue(results.next());
    assertEquals("Don Quixote", results.getString(1));
    assertEquals(1072, results.getInt(2));
    SQLException e = expectThrows(SQLException.class, () -> results.getInt(1));
    assertTrue(e.getMessage(), e.getMessage().contains("unable to convert column 1 to an int"));
    assertFalse(results.next());
}
```



