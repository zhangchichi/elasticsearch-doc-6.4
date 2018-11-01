# 比较运算符

Elasticsearch SQL 支持下列比较运算符：

* 等于（=）

```
SELECT last_name l FROM "test_emp" WHERE emp_no = 10000 LIMIT 5;
```

* 不等（&lt;&gt;  or !=  or &lt;=&gt;）

```
SELECT last_name l FROM "test_emp" WHERE emp_no <> 10000 ORDER BY emp_no LIMIT 5;
```

* 比较

```
SELECT last_name l FROM "test_emp" WHERE emp_no < 10003 ORDER BY emp_no LIMIT 5;
```

* 区间

```
SELECT last_name l FROM "test_emp" WHERE emp_no BETWEEN 9990 AND 10003 ORDER BY emp_no;

```

* 为空/不为空

```
SELECT last_name l FROM "test_emp" WHERE emp_no IS NOT NULL AND gender IS NULL;
```







