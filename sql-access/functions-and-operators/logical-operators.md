# 逻辑运算符

Elasticsearch SQL 支持下列逻辑运算符：

* and

```
SELECT last_name l FROM "test_emp" WHERE emp_no > 10000 AND emp_no < 10005 ORDER BY emp_no LIMIT 5;
```

* or

```
SELECT last_name l FROM "test_emp" WHERE emp_no < 10003 OR emp_no = 10005 ORDER BY emp_no LIMIT 5;
```

* not

```
SELECT last_name l FROM "test_emp" WHERE NOT emp_no = 10000 LIMIT 5;
```



