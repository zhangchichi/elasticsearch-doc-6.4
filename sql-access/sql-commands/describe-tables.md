# DESCRIBE TABLE

**概要**

```
DESCRIBE table
```

或者

```
DESC table
```

**描述**

`DESC` and `DESCRIBE` 是 [SHOW COLUMNS](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-syntax-show-columns.html) 的别名

```
DESCRIBE emp;

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

.

