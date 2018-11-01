# SHOW FUNCTIONS

### 概要

```
SHOW FUNCTIONS [ LIKE? pattern? ]?

pattern
    SQL 匹配模式
```

描述：列出所有SQL函数及其类型。 LIKE子句可用于将名称列表限制为给定模式。

```
SHOW FUNCTIONS;

      name      |     type
----------------+---------------
AVG             |AGGREGATE
COUNT           |AGGREGATE
MAX             |AGGREGATE
MIN             |AGGREGATE
SUM             |AGGREGATE
STDDEV_POP      |AGGREGATE
VAR_POP         |AGGREGATE
PERCENTILE      |AGGREGATE
PERCENTILE_RANK |AGGREGATE
SUM_OF_SQUARES  |AGGREGATE
SKEWNESS        |AGGREGATE
KURTOSIS        |AGGREGATE
DAY_OF_MONTH    |SCALAR
DAY             |SCALAR
DOM             |SCALAR
DAY_OF_WEEK     |SCALAR
DOW             |SCALAR
DAY_OF_YEAR     |SCALAR
DOY             |SCALAR
HOUR_OF_DAY     |SCALAR
HOUR            |SCALAR
MINUTE_OF_DAY   |SCALAR
MINUTE_OF_HOUR  |SCALAR
MINUTE          |SCALAR
SECOND_OF_MINUTE|SCALAR
SECOND          |SCALAR
MONTH_OF_YEAR   |SCALAR
MONTH           |SCALAR
YEAR            |SCALAR
WEEK_OF_YEAR    |SCALAR
WEEK            |SCALAR
ABS             |SCALAR
ACOS            |SCALAR
ASIN            |SCALAR
ATAN            |SCALAR
ATAN2           |SCALAR
CBRT            |SCALAR
CEIL            |SCALAR
CEILING         |SCALAR
COS             |SCALAR
COSH            |SCALAR
COT             |SCALAR
DEGREES         |SCALAR
E               |SCALAR
EXP             |SCALAR
EXPM1           |SCALAR
FLOOR           |SCALAR
LOG             |SCALAR
LOG10           |SCALAR
MOD             |SCALAR
PI              |SCALAR
POWER           |SCALAR
RADIANS         |SCALAR
RANDOM          |SCALAR
RAND            |SCALAR
ROUND           |SCALAR
SIGN            |SCALAR
SIGNUM          |SCALAR
SIN             |SCALAR
SINH            |SCALAR
SQRT            |SCALAR
TAN             |SCALAR
ASCII           |SCALAR
CHAR            |SCALAR
BIT_LENGTH      |SCALAR
CHAR_LENGTH     |SCALAR
LCASE           |SCALAR
LENGTH          |SCALAR
LTRIM           |SCALAR
RTRIM           |SCALAR
SPACE           |SCALAR
UCASE           |SCALAR
SCORE           |SCORE
```

返回的函数列表可以根据模式进行自定义。

可以是精准匹配：

```
SHOW FUNCTIONS LIKE 'ABS';

     name      |     type
---------------+---------------
ABS            |SCALAR
```

可以通配符匹配一个字符：

```
SHOW FUNCTIONS LIKE 'A__';

     name      |     type
---------------+---------------
AVG            |AGGREGATE
ABS            |SCALAR
```

可以匹配大于0个字符：

```
SHOW FUNCTIONS LIKE 'A%';

     name      |     type
---------------+---------------
AVG            |AGGREGATE
ABS            |SCALAR
ACOS           |SCALAR
ASIN           |SCALAR
ATAN           |SCALAR
ATAN2          |SCALAR
ASCII          |SCALAR
```

或者，上面的变化：

```
SHOW FUNCTIONS '%DAY%';

     name      |     type
---------------+---------------
DAY_OF_MONTH   |SCALAR
DAY            |SCALAR
DAY_OF_WEEK    |SCALAR
DAY_OF_YEAR    |SCALAR
HOUR_OF_DAY    |SCALAR
MINUTE_OF_DAY  |SCALAR
```



