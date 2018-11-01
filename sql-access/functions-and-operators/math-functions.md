# 数学方程

所有数学和三角函数都要求其输入（如果适用）为数字。

### 通用

* `ABS`

绝对值，返回\ \[与输入相同的类型\]

```
SELECT ABS(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* `CBRT`

立方根，返回`double`

* `CEIL`

向上取余，返回 `double`

* `CEILING`

同 `CEIL`

* `E`

欧拉数, 返回`2.7182818284590452354`

* [Round](https://en.wikipedia.org/wiki/Rounding#Round_half_up)

这轮“半上升”意味着ROUND（-1.5）导致-1。

* [Floor](https://en.wikipedia.org/wiki/Floor_and_ceiling_functions)
  向下取整

* [Natural logarithm](https://en.wikipedia.org/wiki/Natural_logarithm) （log）
  自然对数

```
SELECT LOG(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Logarithm](https://en.wikipedia.org/wiki/Logarithm) 
  10为底对数（LOG10）

```
SELECT LOG10(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Square root](https://en.wikipedia.org/wiki/Square_root) \(`SQRT`\) 平方根

```
SELECT SQRT(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [e](https://en.wikipedia.org/wiki/Exponential_function)[^1]\(`EXP`\) 指数

```
SELECT EXP(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [e](https://docs.oracle.com/javase/8/docs/api/java/lang/Math.html#expm1-double-)[^2] -1 \(`EXPM1`\)指数

```
SELECT EXP(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

### 三角函数

* Convert from [radians](https://en.wikipedia.org/wiki/Radian) to [degrees](https://en.wikipedia.org/wiki/Degree_%28angle%29) \(`DEGREES`\)

```
SELECT DEGREES(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* Convert from [degrees](https://en.wikipedia.org/wiki/Degree_%28angle%29) to [radians](https://en.wikipedia.org/wiki/Radian) \(`RADIANS`\)

```
SELECT RADIANS(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Sine](https://en.wikipedia.org/wiki/Trigonometric_functions#sine)\(`SIN`\)

```
SELECT SIN(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Cosine](https://en.wikipedia.org/wiki/Trigonometric_functions#cosine)\(`COS`\)

```
SELECT COS(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Tangent](https://en.wikipedia.org/wiki/Trigonometric_functions#tangent) \(`TAN`\)

```
SELECT TAN(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Arc sine](https://en.wikipedia.org/wiki/Inverse_trigonometric_functions) \(`ASIN`\)

```
SELECT ASIN(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Arc cosine](https://en.wikipedia.org/wiki/Inverse_trigonometric_functions)\(`ACOS`\)

```
SELECT ACOS(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Arc tangent](https://en.wikipedia.org/wiki/Inverse_trigonometric_functions)\(`ATAN`\)

```
SELECT ATAN(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Hyperbolic sine](https://en.wikipedia.org/wiki/Hyperbolic_function)\(`SINH`\)

```
SELECT SINH(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```

* [Hyperbolic cosine](https://en.wikipedia.org/wiki/Hyperbolic_function)\(`COSH`\)

```
SELECT COSH(emp_no) m, first_name FROM "test_emp" WHERE emp_no < 10010 ORDER BY emp_no;
```





[^1]: x

[^2]: x

