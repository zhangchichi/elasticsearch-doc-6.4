# SHOW TABLES

### 概要

```
SHOW TABLES [ LIKE? pattern? ]?

pattern：sql模式匹配
```

描述：列出当前用户可用的表及其类型。

```
SHOW TABLES;

     name      |     type
---------------+---------------
emp            |BASE TABLE
employees      |ALIAS
library        |BASE TABLE
```

LIKE子句可用于将名称列表限制为给定模式。

可以精确匹配：

```
SHOW TABLES LIKE 'emp';

     name      |     type
---------------+---------------
emp            |BASE TABLE
```

多字符：

```
SHOW TABLES LIKE 'emp%';

     name      |     type
---------------+---------------
emp            |BASE TABLE
employees      |ALIAS
```

单字符：

```
SHOW TABLES LIKE 'em_';

     name      |     type
---------------+---------------
emp            |BASE TABLE
```

或者混用：

```
SHOW TABLES LIKE '%em_';

     name      |     type
---------------+---------------
emp            |BASE TABLE
```



