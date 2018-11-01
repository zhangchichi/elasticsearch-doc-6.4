# 集合方程

### 基础

* [Average](https://en.wikipedia.org/wiki/Arithmetic_mean) （AVG）平均数

```
SELECT AVG(salary) AS avg FROM test_emp;
```

* 计算匹配字段的数量 \(`COUNT`\)

```
SELECT COUNT(*) AS count FROM test_emp;
```

* 计算匹配文档中不同值的数量 \(`COUNT(DISTINCT`\)

```
SELECT COUNT(DISTINCT hire_date) AS count FROM test_emp;
```

* 在匹配文档中找最大值 \(`MAX`\)

```
SELECT MAX(salary) AS max FROM test_emp;
```

* 在匹配文档中找最小值 \(`MIN`\)

```
SELECT MIN(emp_no) AS min FROM test_emp;
```

* [求和](https://en.wikipedia.org/wiki/Kahan_summation_algorithm) \(`SUM`\).

```
SELECT SUM(salary) FROM test_emp;
```



