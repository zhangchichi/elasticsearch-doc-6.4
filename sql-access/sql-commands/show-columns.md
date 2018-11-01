# SHOW COLUMNS

### 概要

```
SHOW COLUMNS [ FROM | IN ] ? table
```

描述：列出表中的列及其数据类型（和其他属性）。

```
SHOW COLUMNS IN emp;

       column       |     type
--------------------+---------------
birth_date          |TIMESTAMP
dep                 |STRUCT
dep.dep_id          |VARCHAR
dep.dep_name        |VARCHAR
dep.dep_name.keyword|VARCHAR
dep.from_date       |TIMESTAMP
dep.to_date         |TIMESTAMP
emp_no              |INTEGER
first_name          |VARCHAR
first_name.keyword  |VARCHAR
gender              |VARCHAR
hire_date           |TIMESTAMP
languages           |TINYINT
last_name           |VARCHAR
last_name.keyword   |VARCHAR
salary              |INTEGER
```



