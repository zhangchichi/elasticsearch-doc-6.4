# SELECT

### 概要

```
SELECT select_expr [, ...]
[ FROM table_name ]
[ WHERE condition ]
[ GROUP BY grouping_element [, ...] ]
[ HAVING condition]
[ ORDER BY expression [ ASC | DESC ] [, ...] ]
[ LIMIT [ count ] ]
```

**描述**：从零个或多个表中检索行。

SELECT的一般执行如下：

1. 计算FROM列表中的所有元素（每个元素可以是基表或别名表）。目前，FROM仅支持一个表。但请注意，表名可以是一种模式（参见下面的FROM子句）。
2. 如果指定了WHERE子句，则从输出中消除所有不满足条件的行。 （参见下面的WHERE子句。）
3. 如果指定了GROUP BY子句，或者存在聚合函数调用，则将输出组合成与一个或多个值匹配的行组，并计算聚合函数的结果。如果存在HAVING子句，则会删除不满足给定条件的组。 （参见下面的GROUP BY子句和HAVING子句。）
4. 使用每个选定行或行组的SELECT输出表达式计算实际输出行。
5. 如果指定了ORDER BY子句，则返回的行按指定的顺序排序。如果未给出ORDER BY，则以系统发现最快生成的顺序返回行。 （参见下面的ORDER BY子句。）
6. 如果指定了LIMIT，则SELECT语句仅返回结果行的子集。 （参见下面的LIMIT子句。）

### select list

SELECT列表，即SELECT和FROM之间的表达式，表示SELECT语句的输出行。

与表一样，SELECT的每个输出列都有一个名称，可以通过AS关键字为每列指定：

```
SELECT 1 + 1 AS result

    result
---------------
2
```

注意：AS是一个可选的关键字，但它有助于提高可读性，并且在某些情况下有助于查询的模糊性，这也是建议指定它的原因。

如果没有给出名称，则由Elasticsearch SQL分配：

```
SELECT 1 + 1;

    (1 + 1)
---------------
2
```

或者如果它是一个简单的列引用，请使用其名称作为列名：

```
SELECT emp_no FROM emp LIMIT 1;

    emp_no
---------------
10001
```

### 通配符

查询远中的所有列，使用 \*：

```
SELECT * FROM emp LIMIT 1;

     birth_date     |    emp_no     |  first_name   |    gender     |     hire_date      |   languages   |   last_name   |    salary
--------------------+---------------+---------------+---------------+--------------------+---------------+---------------+---------------
1953-09-02T00:00:00Z|10001          |Georgi         |M              |1986-06-26T00:00:00Z|2              |Facello        |57305
```

它基本上返回所有（顶级字段，子字段，如多字段被忽略‘multi-fields are ignored’）列。

### FROM子句

FROM子句为SELECT指定一个表，并具有以下语法：

```
FROM table_name [ [ AS ] alias ]
```

```
table_name 
    表示现有表的名称（可选地限定），可以是具体的或基本的（实际索引）或别名。
```

如果表名包含特殊的SQL字符（例如。， - 等等），请使用双引号来转义它们：

```
SELECT * FROM "emp" LIMIT 1;

     birth_date     |    emp_no     |  first_name   |    gender     |     hire_date      |   languages   |   last_name   |    salary
--------------------+---------------+---------------+---------------+--------------------+---------------+---------------+---------------
1953-09-02T00:00:00Z|10001          |Georgi         |M              |1986-06-26T00:00:00Z|2              |Facello        |57305
```

名称可以是指向多个索引的模式（可能需要如上所述引用），其限制是所有已解析的具体表都具有精确映射。

```
SELECT emp_no FROM "e*p" LIMIT 1;

    emp_no
---------------
10001
```

```
alias 
    包含别名的FROM项的替换名称。 别名用于简洁或消除歧义。 提供别名时，它会完全隐藏表的实际名称，并且必须在其位置使用。
```

```
SELECT e.emp_no FROM emp AS e LIMIT 1;

    emp_no
-------------
10001
```

### WHERE子句

可选的WHERE子句用于从查询中过滤行，并具有以下语法：

```
WHERE condition
```

```
condition
    表示求值为布尔值的表达式。 仅返回与条件匹配的行（为true）。
```

```
SELECT last_name FROM emp WHERE emp_no = 10001;

   last_name
---------------
Facello
```

### GROUP BY

GROUP BY子句用于将结果划分为来自指定列的匹配值的行组。 它具有以下语法：

```
GROUP BY grouping_element [, ...]
```

```
grouping_element  
    表示正在搜索行的表达式。 它可以是列的列名，别名或序号或列值的任意表达式。
```

一个常见的分组名称：

```
SELECT gender AS g FROM emp GROUP BY gender;

       g
---------------
F
M
```

输出顺序分组：

```
SELECT gender FROM emp GROUP BY 1;

    gender
---------------
F
M
```

别名分组：

```
SELECT gender AS g FROM emp GROUP BY g;

       g
---------------
F
M
```

列表达式分组（与别名一起使用）：

```
SELECT languages + 1 AS l FROM emp GROUP BY l;

       l
---------------
2
3
4
5
6
```

或者混合上面的：

```
SELECT gender g, languages l, COUNT(*) c FROM "emp" GROUP BY g, l ORDER BY languages ASC, gender DESC;

       g       |       l       |       c
---------------+---------------+---------------
F              |2              |4
F              |3              |8
F              |4              |7
F              |5              |7
F              |6              |11
M              |2              |12
M              |3              |12
M              |4              |15
M              |5              |11
M              |6              |13
```

在SELECT中使用GROUP BY子句时，所有输出表达式必须是聚合函数或用于分组或派生的表达式（否则每个未分组列将返回多个可能的值）。

```
SELECT gender AS g, COUNT(*) AS c FROM emp GROUP BY gender;

       g       |       c
---------------+---------------
F              |37
M              |63
```

输出表达式中使用聚合：

```
SELECT gender AS g, ROUND(MIN(salary) / 100) AS salary FROM emp GROUP BY gender;

       g       |    salary
---------------+---------------
F              |260
M              |253

```

使用多聚合：

```
SELECT gender AS g, KURTOSIS(salary) AS k, SKEWNESS(salary) AS s FROM emp GROUP BY gender;

       g       |        k         |         s
---------------+------------------+-------------------
F              |1.8427808415250482|0.04517149340491813
M              |2.259327644285826 |0.40268950715550333
```

### 隐式分组

如果在没有关联的GROUP BY的情况下使用聚合，则会应用隐式分组，这意味着所有选定的行都被视为形成单个默认或隐式组。 因此，查询仅发出一行（因为只有一个组）。

一个常见的例子是计算记录数：

```
SELECT COUNT(*) AS count FROM emp;

     count
---------------
100
```

当然也适用于多个集合：

```
SELECT MIN(salary) AS min, MAX(salary) AS max, AVG(salary) AS avg, COUNT(*) AS count FROM emp;

      min      |      max      |      avg      |     count
---------------+---------------+---------------+---------------
25324          |74999          |48248          |100
```

### HAVING

HAVING子句只能用于聚合函数（以及GROUP BY）来过滤保留或不保留的组，并具有以下语法：

```
GROUP BY condition
```

```
condition
    表示求值为布尔值的表达式。 仅返回与条件匹配的组（为true）。
```

WHERE和HAVING都用于过滤，但它们之间存在几个显着差异：

1. WHERE适用于各个行，HAVING适用于由\`\`GROUP BY\`\`创建的组
2. WHERE再分组前执行，HAVING再分组后执行

```
SELECT languages AS l, COUNT(*) AS c FROM emp GROUP BY l HAVING c BETWEEN 15 AND 20;

       l       |       c
---------------+---------------
1              |16
2              |20
4              |18
```

此外，可以在HAVING中使用多个聚合表达式，甚至可以在输出中未使用的表达式（SELECT）：

```
SELECT MIN(salary) AS min, MAX(salary) AS max, MAX(salary) - MIN(salary) AS diff FROM emp GROUP BY languages HAVING diff - max % min > 0 AND AVG(salary) > 30000;

      min      |      max      |     diff
---------------+---------------+---------------
25976          |73717          |47741
29175          |73578          |44403
26436          |74999          |48563
27215          |74572          |47357
25324          |73851          |48527
```

### 隐式分组

如上所述，可以有一个没有“GROUP BY”的HAVING子句。 在这种情况下，应用所谓的隐式分组，这意味着所有选定的行都被视为形成一个组，并且HAVING可以应用于此组中指定的任何聚合函数。 \`因此，查询只发出一行（因为只有一个组），HAVING条件返回一行（组）或者如果条件失败则返回零。

在此示例中，HAVING匹配到了数据：

```
SELECT MIN(salary) AS min, MAX(salary) AS max FROM emp HAVING min > 25000;

      min      |      max
---------------+---------------
25324          |74999
```

### ORDER BY

