# 日期和时间方程

* 日期中提取年份 \(`YEAR`\)

```
SELECT YEAR(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS year;

     year
---------------
2018
```

* 日期中提取月份 \(`MONTH_OF_YEAR`or`MONTH`\)

```
SELECT MONTH_OF_YEAR(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS month;

     month
---------------
2
```

* 日期中提取周 \(`WEEK_OF_YEAR`or`WEEK`\)

```
SELECT WEEK_OF_YEAR(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS week;

     week
---------------
8
```

* 日期中提取年的天数 \(`DAY_OF_YEAR`or`DOY`\)

```
SELECT DAY_OF_YEAR(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS day;

      day
---------------
50
```

* 日期中提取月的天数 \(`DAY_OF_MONTH`,`DOM`, or`DAY`\)

```
SELECT DAY_OF_MONTH(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS day;

      day
---------------
19
```

* 日期中提取周的天数 \(`DAY_OF_WEEK`or`DOW`\).周一是`1`, 周二是`2`, 等.

```
SELECT DAY_OF_WEEK(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS day;

      day
---------------
1
```

* 日期中提取一天的小时数\(`HOUR_OF_DAY`or`HOUR`\). 

```
SELECT HOUR_OF_DAY(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS hour;

     hour
---------------
10
```

* 日期中提取一天的分钟数 \(`MINUTE_OF_DAY`\).

```
SELECT MINUTE_OF_DAY(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS minute;

    minute
---------------
623
```

* 日期中提取一个小时的分钟数 \(`MINUTE_OF_HOUR`,`MINUTE`\).

```
SELECT MINUTE_OF_HOUR(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS minute;

    minute
---------------
23
```

* 日期中提取分钟的毫秒数 \(`SECOND_OF_MINUTE`,`SECOND`\).

```
SELECT SECOND_OF_MINUTE(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS second;

    second
---------------
27
```

* Extract

作为替代方案，可以支持EXTRACT从日期时间中提取字段。 您可以使用EXTRACT（&lt;datetime\_function&gt; FROM &lt;expression&gt;）运行任何日期时间函数。 所以

```
SELECT EXTRACT(DAY_OF_YEAR FROM CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS day;

      day
---------------
50
```

等价于

```
SELECT DAY_OF_YEAR(CAST('2018-02-19T10:23:27Z' AS TIMESTAMP)) AS day;

      day
---------------
50
```



