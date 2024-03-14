> *浅浅记录一下MySQL的基本操作，以用来备忘。这里直接从DQL语言开始，因为在基本操作中，查询才是重中之重，至与优化，会有其他文章进行记录。*

## SQL分类

SQL语言在功能上主要分为如下3大类： **DDL**（`Data Definition Languages`、数据定义语言），这些语句定义了不同的数据库、表、视图、索引等数据库对象，还可以用来创建、删除、修改数据库和数据表的结构。主要的语句关键字包括 CREATE 、 DROP 、 ALTER 等。

**DML**（`Data Manipulation Language`、数据操作语言），用于添加、删除、更新和查询数据库记录，并检查数据完整性。主要的语句关键字包括 INSERT 、 DELETE 、 UPDATE 、 SELECT 等。SELECT是SQL语言的基础，最为重要。

**DCL**（`Data Control Language`、数据控制语言），用于定义数据库、表、字段、用户的访问权限和安全级别。主要的语句关键字包括 GRANT 、 REVOKE 、 COMMIT、ROLLBACK 、 SAVEPOINT 等。

因为查询语句使用的非常的频繁，所以很多人把查询语句单拎出来一类：DQL（数据查询语言）。还有单独将 COMMIT 、 ROLLBACK 取出来称为TCL （Transaction Control Language，事务控制语言）。

## 基本的SELECT语句

> *MySQL中的SQL语句是不区分大小写的，因此SELECT和select的作用是相同的，但是，许多开发人员习惯将关键字大写、数据列和表名小写，读者也应该养成一个良好的编程习惯，这样写出来的代码更容易阅读和维护。*

#### 0 SELECT…

```SQL
SELECT 1; #没有任何子句
SELECT 9/2; #没有任何子句
```

#### SELECT … FROM

```SQL
SELECT 标识选择哪些列
FROM 标识从哪个表中选择
```

查询学生表全部信息

```SQL
查询学生表全部信息
SELECT * FROM user;
```

查询学生表制定列

```SQL
SELECT name FROM user;
```

#### 给列起别名

查询部门信息，并起别名。

```SQL
SELECT last_name AS name, commission_pct comm
FROM employees;
```

#### 去除重复行

默认情况下，查询会返回全部行，包括重复行。这个时候我们需要关键字`distinct `去重。

```SQL
SELECT DISTINCT department_id,salary
FROM employees;
```

`DISTINCT `需要放到所有列名的前面，如果写成 SELECT salary, DISTINCT department_id FROM employees 会报错。DISTINCT 其实是对后面所有列名的组合进行去重。

#### 空值参与运算

所有运算符或列值遇到null值，运算的结果都为null

```SQL
SELECT employee_id,salary,commission_pct,
12 * salary * (1 + commission_pct) "annual_sal"
FROM employees;
```

输出

```SQL
+-------------+----------+----------------+------------+
| employee_id | salary   | commission_pct | annual_sal |
+-------------+----------+----------------+------------+
|         100 | 24000.00 |           NULL |       NULL |
|         101 | 17000.00 |           NULL |       NULL |
|         102 | 17000.00 |           NULL |       NULL |
|         103 |  9000.00 |           NULL |       NULL |
|         104 |  6000.00 |           NULL |       NULL |
|         105 |  4800.00 |           NULL |       NULL |
|         106 |  4800.00 |           NULL |       NULL |
|         107 |  4200.00 |           NULL |       NULL |
———————————————————————————————————————————————————————+
```

这里一定要注意，在 MySQL 里面， 空值不等于空字符串。一个空字符串的长度是 0，而一个空值的长度是空。而且，在 MySQL 里面，空值是占用空间的。null不等同于0、’ ’ 以及’null’.

#### 查询常数

SELECT 查询还可以对常数进行查询。对的，就是在 SELECT 查询结果中增加一列固定的常数列。这列的取值是我们指定的，而不是从数据表中动态取出的。 你可能会问为什么我们还要对常数进行查询呢？ SQL 中的 SELECT 语法的确提供了这个功能，一般来说我们只从一个表中查询数据，通常不需要增加一个固定的常数列，但如果我们想整合不同的数据源，用常数列作为这个表的标记，就需要查询常数。比如说，我们想对 employees 数据表中的员工姓名进行查询，同时增加一列字段 corporation ，这个字段固定值为“尚硅谷”，可以这样写：

```SQL
SELECT '山竹' as corporation, last_name FROM employees;
```

#### 显示表结构

使用DESCRIBE 或 DESC 命令，表示表结构。

```SQL
DESCRIBE employees;
或
DESC employees;
```

其中，各个字段的含义分别解释如下： Field：表示字段名称。 Type：表示字段类型，这里 barcode、goodsname 是文本型的，price 是整数类型的。 Null：表示该列是否可以存储NULL值。 Key：表示该列是否已编制索引。PRI表示该列是表主键的一部分；UNI表示该列是UNIQUE索引的一部分；MUL表示在列中某个给定值允许出现多次。 Default：表示该列是否有默认值，如果有，那么值是多少。 Extra：表示可以获取的与给定列有关的附加信息，例如AUTO_INCREMENT等。

#### 条件查询

使用WHERE 子句，将不满足条件的行过滤掉WHERE子句紧随 FROM子句

```SQL
SELECT 字段1,字段2
FROM 表名
WHERE 过滤条件
```

## 运算符

### 算术运算符

算术运算符主要用于数学运算，其可以连接运算符前后的两个数值或表达式，对数值或表达式进行加（+）、减（-）、乘（*）、除

#### 加法与减法运算符

```SQL
SELECT 100, 100 + 0, 100 - 0, 100 + 50, 100 + 50 -30, 100 + 35.5, 100 - 35.5
FROM dual;
/*输出：
+-----+---------+---------+----------+--------------+------------+------------+
| 100 | 100 + 0 | 100 - 0 | 100 + 50 | 100 + 50 -30 | 100 + 35.5 | 100 - 35.5 |
+-----+---------+---------+----------+--------------+------------+------------+
| 100 |     100 |     100 |      150 |          120 |      135.5 |       64.5 |
+-----+---------+---------+----------+--------------+------------+------------+
*/
```

由运算结果可以得出如下结论：

- 一个整数类型的值对整数进行加法和减法操作，结果还是一个整数；
- 一个整数类型的值对浮点数进行加法和减法操作，结果是一个浮点数；
- 加法和减法的优先级相同，进行先加后减操作与进行先减后加操作的结果是一样的；
- 在Java中，+的左右两边如果有字符串，那么表示字符串的拼接。但是在MySQL中+只表示数值相加。如果遇到非数值类型，先尝试转成数值，如果转失败，就按0计算。（补充：MySQL中字符串拼接要使用字符串函数CONCAT()实现）

#### 乘法与除法运算符

```SQL
SELECT 100, 100 * 1, 100 * 1.0, 100 / 1.0, 100 / 2,100 + 2 * 5 / 2,100 /3, 100 DIV 0 FROM dual;
/*输出：
+-----+---------+-----------+-----------+---------+-----------------+---------+-----------+
| 100 | 100 * 1 | 100 * 1.0 | 100 / 1.0 | 100 / 2 | 100 + 2 * 5 / 2 | 100 /3  | 100 DIV 0 |
+-----+---------+-----------+-----------+---------+-----------------+---------+-----------+
| 100 |     100 |     100.0 |  100.0000 | 50.0000 |        105.0000 | 33.3333 |      NULL |
+-----+---------+-----------+-----------+---------+-----------------+---------+-----------+
*/
#计算出员工的年基本工资
SELECT employee_id,salary,salary * 12 annual_sal FROM employees;
/*部分输出
+-------------+----------+------------+
| employee_id | salary   | annual_sal |
+-------------+----------+------------+
|         100 | 24000.00 |  288000.00 |
|         101 | 17000.00 |  204000.00 |
|         102 | 17000.00 |  204000.00 |
|         103 |  9000.00 |  108000.00 |
|         104 |  6000.00 |   72000.00 |
|         105 |  4800.00 |   57600.00 |
*/
```

由运算结果可以得出如下结论：

- 一个数乘以整数1和除以整数1后仍得原数；
- 一个数乘以浮点数1和除以浮点数1后变成浮点数，数值与原数相等；
- 一个数除以整数后，不管是否能除尽，结果都为一个浮点数；
- 一个数除以另一个数，除不尽时，结果为一个浮点数，并保留到小数点后4位；
- 乘法和除法的优先级相同，进行先乘后除操作与先除后乘操作，得出的结果相同。
- 在数学运算中，0不能用作除数，在MySQL中，一个数除以0为NULL。

#### 求模（求余）运算符

```SQL
SELECT 12 % 3, 12 MOD 5 FROM dual;
/*
+--------+----------+
| 12 % 3 | 12 MOD 5 |
+--------+----------+
|      0 |        2 |
+--------+----------+
*/
#筛选出employee_id是偶数的员工
SELECT * FROM employees WHERE employee_id MOD 2 = 0;
/*
+-------------+-------------+-------------+----------+--------------------+------------+------------+----------+----------------+------------+---------------+
| employee_id | first_name  | last_name   | email    | phone_number       | hire_date  | job_id     | salary   | commission_pct | manager_id | department_id |
+-------------+-------------+-------------+----------+--------------------+------------+------------+----------+----------------+------------+---------------+
|         100 | Steven      | King        | SKING    | 515.123.4567       | 1987-06-17 | AD_PRES    | 24000.00 |           NULL |       NULL |            90 |
|         102 | Lex         | De Haan     | LDEHAAN  | 515.123.4569       | 1993-01-13 | AD_VP      | 17000.00 |           NULL |        100 |            90 |
*/
```

### 比较运算符

比较运算符用来对表达式左边的操作数和右边的操作数进行比较，比较的结果为真则返回1，比较的结果为假则返回0，其他情况则返回NULL。

比较运算符经常被用来作为SELECT查询语句的条件来使用，返回符合条件的结果记录。

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/230f16f8fba44c6c8d95ae35e4c47499.png)

#### 等号运算符

等号运算符（=）判断等号两边的值、字符串或表达式是否相等，如果相等则返回1，不相等则返回0。

在使用等号运算符时，遵循如下规则：

1.如果等号两边的值、字符串或表达式都为字符串，则MySQL会按照字符串进行比较，其比较的是每个字符串中字符的ANSI编码是否相等。

2.如果等号两边的值都是整数，则MySQL会按照整数来比较两个值的大小。

3.如果等号两边的值一个是整数，另一个是字符串，则MySQL会将字符串转化为数字进行比较。

4.如果等号两边的值、字符串或表达式中有一个为NULL，则比较结果为NULL。

#### 安全等于运算符

安全等于运算符（<=>）与等于运算符（=）的作用是相似的， 唯一的区别 是‘<=>’可以用来对NULL进行判断。在两个操作数均为NULL时，其返回值为1，而不为NULL；当一个操作数为NULL时，其返回值为0，而不为NULL。

#### 不等于运算符

不等于运算符（<>和!=）用于判断两边的数字、字符串或者表达式的值是否不相等，如果不相等则返回1，相等则返回0。不等于运算符不能判断NULL值。如果两边的值有任意一个为NULL，或两边都为NULL，则结果为NULL。 SQL语句示例如下：

```SQL
 SELECT 1 <> 1, 1 != 2, 'a' != 'b', (3+4) <> (2+6), 'a' != NULL, NULL <> NULL;
 /*
+--------+--------+------------+----------------+-------------+--------------+
| 1 <> 1 | 1 != 2 | 'a' != 'b' | (3+4) <> (2+6) | 'a' != NULL | NULL <> NULL |
+--------+--------+------------+----------------+-------------+--------------+
|      0 |      1 |          1 |              1 |        NULL |         NULL |
+--------+--------+------------+----------------+-------------+--------------+
*/
```

#### 非符号类型运算符

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/3789e3aca48443a88c42ac1845b14808.png)

#### 空运算符

空运算符（IS NULL或 ISNULL）判断一个值是否为NULL，如果为NULL则返回1，否则返回0。 SQL语句示例如下：

```SQL
 SELECT NULL IS NULL, ISNULL(NULL), ISNULL('a'), 1 IS NULL;
 /*输出
+--------------+--------------+-------------+-----------+
| NULL IS NULL | ISNULL(NULL) | ISNULL('a') | 1 IS NULL |
+--------------+--------------+-------------+-----------+
|            1 |            1 |           0 |         0 |
+--------------+--------------+-------------+-----------+
*/
SELECT employee_id,commission_pct FROM employees WHERE commission_pct IS NULL;
#或
SELECT employee_id,commission_pct FROM employees WHERE commission_pct <=> NULL;
#或
/*输出：
+-------------+----------------+
| employee_id | commission_pct |
+-------------+----------------+
|         100 |           NULL |
|         101 |           NULL |
|         102 |           NULL |
|         103 |           NULL |
|         104 |           NULL |
*/
SELECT employee_id,commission_pct FROM employees WHERE commission_pct = NULL;
#输出Empty set (0.00 sec)
SELECT last_name, manager_id FROM employees WHERE manager_id IS NULL;
/*输出
SELECT last_name, manager_id FROM employees WHERE manager_id IS NULL;
+-----------+------------+
| last_name | manager_id |
+-----------+------------+
| King      |       NULL |
+-----------+------------+
*/
```

#### 非空运算符

非空运算符（IS NOT NULL）判断一个值是否不为NULL，如果不为NULL则返回1，否则返回0。 SQL语句示例如下：

```SQL
 SELECT NULL IS NOT NULL, 'a' IS NOT NULL, 1 IS NOT NULL;
 /*输出
+------------------+-----------------+---------------+
| NULL IS NOT NULL | 'a' IS NOT NULL | 1 IS NOT NULL |
+------------------+-----------------+---------------+
|                0 |               1 |             1 |
+------------------+-----------------+---------------+
*/
SELECT employee_id,commission_pct FROM employees WHERE NOT commission_pct <=> NULL;
#或
SELECT employee_id,commission_pct FROM employees WHERE NOT ISNULL(commission_pct);
/*输出：
+-------------+----------------+
| employee_id | commission_pct |
+-------------+----------------+
|         145 |           0.40 |
|         146 |           0.30 |
|         147 |           0.30 |
|         148 |           0.30 |
|         149 |           0.20 |
|         150 |           0.30 |
|         151 |           0.25 |
|         152 |           0.25 |
*/
```

#### 最小值运算符

语法格式为：LEAST(值1，值2，…，值n)。其中，“值n”表示参数列表中有n个值。在有两个或多个参数的情况下，返回最小值。

```SQL
SELECT LEAST (1,0,2), LEAST('b','a','c'), LEAST(1,NULL,2);
/*输出：
+---------------+--------------------+-----------------+
| LEAST (1,0,2) | LEAST('b','a','c') | LEAST(1,NULL,2) |
+---------------+--------------------+-----------------+
|             0 | a                  |            NULL |
+---------------+--------------------+-----------------+
*/
```

由结果可以看到，当参数是整数或者浮点数时，LEAST将返回其中最小的值；当参数为字符串时，返回字母表中顺序最靠前的字符；当比较值列表中有NULL时，不能判断大小，返回值为NULL

#### 最大值运算符

语法格式为：GREATEST(值1，值2，…，值n)。其中，n表示参数列表中有n个值。当有两个或多个参数时，返回值为最大值。假如任意一个自变量为NULL，则GREATEST()的返回值为NULL。

```SQL
 SELECT GREATEST(1,0,2), GREATEST('b','a','c'), GREATEST(1,NULL,2);
 /*输出
+-----------------+-----------------------+--------------------+
| GREATEST(1,0,2) | GREATEST('b','a','c') | GREATEST(1,NULL,2) |
+-----------------+-----------------------+--------------------+
|               2 | c                     |               NULL |
+-----------------+-----------------------+--------------------+
*/
```

由结果可以看到，当参数中是整数或者浮点数时，GREATEST将返回其中最大的值；当参数为字符串时，返回字母表中顺序最靠后的字符；当比较值列表中有NULL时，不能判断大小，返回值为NULL。

#### BETWEEN AND运算符

BETWEEN运算符使用的格式通常为SELECT D FROM TABLE WHERE C BETWEEN A AND B，此时，当C大于或等于A，并且C小于或等于B时，结果为1，否则结果为0。

```SQL
SELECT 1 BETWEEN 0 AND 1, 10 BETWEEN 11 AND 12, 'b' BETWEEN 'a' AND 'c';
/*输出
+-------------------+----------------------+-------------------------+
| 1 BETWEEN 0 AND 1 | 10 BETWEEN 11 AND 12 | 'b' BETWEEN 'a' AND 'c' |
+-------------------+----------------------+-------------------------+
|                 1 |                    0 |                       1 |
+-------------------+----------------------+-------------------------+
*/
SELECT last_name, salary FROM employees WHERE salary BETWEEN 2500 AND 3500;
 /*输出
 +-------------+---------+
| last_name   | salary  |
+-------------+---------+
| Khoo        | 3100.00 |
| Baida       | 2900.00 |
| Tobias      | 2800.00 |
| Himuro      | 2600.00 |
| Colmenares  | 2500.00 |
| Nayer       | 3200.00 |
| Mikkilineni | 2700.00 |
 */
```

#### IN运算符

IN运算符用于判断给定的值是否是IN列表中的一个值，如果是则返回1，否则返回0。如果给定的值为NULL，或者IN列表中存在NULL，则结果为NULL。

```SQL
SELECT 'a' IN ('a','b','c'), 1 IN (2,3), NULL IN ('a','b'), 'a' IN ('a', NULL);
/*输出：
+----------------------+------------+-------------------+--------------------+
| 'a' IN ('a','b','c') | 1 IN (2,3) | NULL IN ('a','b') | 'a' IN ('a', NULL) |
+----------------------+------------+-------------------+--------------------+
|                    1 |          0 |              NULL |                  1 |
+----------------------+------------+-------------------+--------------------+
*/

 SELECT employee_id, last_name, salary, manager_id FROM employees WHERE manager_id IN (100, 101, 201);
 /*输出
+-------------+-----------+----------+------------+
| employee_id | last_name | salary   | manager_id |
+-------------+-----------+----------+------------+
|         101 | Kochhar   | 17000.00 |        100 |
|         102 | De Haan   | 17000.00 |        100 |
|         108 | Greenberg | 12000.00 |        101 |
|         114 | Raphaely  | 11000.00 |        100 |
*/
```

#### NOT IN运算符

NOT IN运算符用于判断给定的值是否不是IN列表中的一个值，如果不是IN列表中的一个值，则返回1，否则返回0。

```SQL
 SELECT 'a' NOT IN ('a','b','c'), 1 NOT IN (2,3);
 /*输出：
+--------------------------+----------------+
| 'a' NOT IN ('a','b','c') | 1 NOT IN (2,3) |
+--------------------------+----------------+
|                        0 |              1 |
+--------------------------+----------------+
*/
```

#### LIKE运算符

LIKE运算符主要用来匹配字符串，通常用于模糊匹配，如果满足条件则返回1，否则返回0。如果给定的值或者匹配条件为NULL，则返回结果为NULL。LIKE运算符通常使用如下通配符：
 “%”：匹配0个或多个字符。
 “_”：只能匹配一个字符。

```SQL
 SELECT NULL LIKE 'abc', 'abc' LIKE NULL;
 /*输出
+-----------------+-----------------+
| NULL LIKE 'abc' | 'abc' LIKE NULL |
+-----------------+-----------------+
|            NULL |            NULL |
+-----------------+-----------------+
*/
SELECT first_name FROM employees WHERE first_name LIKE 'S%';
/*输出
+------------+
| first_name |
+------------+
| Steven     |
| Shelli     |
| Sigal      |
| Shanta     |
| Steven     |
| Stephen    |
| Sarath     |
| Sundar     |
*/
```

#### ESCAPE

回避特殊符号的：使用转义符。例如：将[%]转为[ $ % ]、[]转为[ $ ]，然后再加上[ESCAPE‘$’]即可。

```SQL
SELECT job_id FROM jobs WHERE job_id LIKE 'IT\_%';
/*输出
+---------+
| job_id  |
+---------+
| IT_PROG |
+---------+
*/

#如果使用\表示转义，要省略ESCAPE。如果不是\，则要加上ESCAPE。
SELECT job_id FROM jobs WHERE job_id LIKE 'IT$_%'  escape '$';
/*输出
+---------+
| job_id  |
+---------+
| IT_PROG |
+---------+
*/
```

#### REGEXP运算符

REGEXP运算符用来匹配字符串，语法格式为： expr REGEXP 匹配条件 如果expr满足匹配条件，返回1；如果不满足，则返回0。若expr或匹配条件任意一个NULL，则结果为NULL。 REGEXP运算符在进行匹配时，常用的有下面几种通配符： （1）‘^’匹配以该字符后面的字符开头的字符串。 （2）‘$’匹配以该字符前面的字符结尾的字符串。 （3）‘.’匹配任何一个单字符。 （4）“[…]”匹配在方括号内的任何字符。例如，“[abc]”匹配“a”或“b”或“c”。为了命名字符的范围，使用一个‘-’。“[a-z]”匹配任何字母，而“[0-9]”匹配任何数字。 （5）‘*’匹配零个或多个在它前面的字符。例如，“x*”匹配任何数量的‘x’字符，“[0-9]*”匹配任何数量的数字，而“*”匹配任何数量的任何字符。

```SQL
SELECT 'shkstart' REGEXP '^s', 'shkstart' REGEXP 't$', 'shkstart' REGEXP 'hk';
/*输出
+------------------------+------------------------+------------------------+
| 'shkstart' REGEXP '^s' | 'shkstart' REGEXP 't$' | 'shkstart' REGEXP 'hk' |
+------------------------+------------------------+------------------------+
|                      1 |                      1 |                      1 |
+------------------------+------------------------+------------------------+
*/

SELECT 'atguigu' REGEXP 'gu.gu', 'atguigu' REGEXP '[ab]';
/*输出
+--------------------------+-------------------------+
| 'atguigu' REGEXP 'gu.gu' | 'atguigu' REGEXP '[ab]' |
+--------------------------+-------------------------+
|                        1 |                       1 |
+--------------------------+-------------------------+
*/
```

### 逻辑运算符

逻辑运算符主要用来判断表达式的真假，在MySQL中，逻辑运算符的返回结果为1、0或者NULL。
 MySQL中支持4种逻辑运算符如下：

![img](https://shanzhu-edu.oss-cn-shanghai.aliyuncs.com/b00c28394337418389f0d60e821c43e2.png)

#### 逻辑非运算符

逻辑非（NOT或!）运算符表示当给定的值为0时返回1；当给定的值为0值时返回1；
 当给定的值为NULL时，返回NULL。

```SQL
 SELECT NOT 1, NOT 0, NOT(1+1), NOT !1, NOT NULL;
 /*输出：
+-------+-------+----------+--------+----------+
| NOT 1 | NOT 0 | NOT(1+1) | NOT !1 | NOT NULL |
+-------+-------+----------+--------+----------+
|     0 |     1 |        0 |      1 |     NULL |
+-------+-------+----------+--------+----------+
*/

 SELECT last_name, job_id FROM employees WHERE job_id NOT IN ('IT_PROG','ST_CLERK', 'SA_REP');
 /*部分输出：
+------------+------------+
| last_name  | job_id     |
+------------+------------+
| King       | AD_PRES    |
| Kochhar    | AD_VP      |
| De Haan    | AD_VP      |
| Greenberg  | FI_MGR     |
*/
```

#### 逻辑与运算符

逻辑与（AND或&&）运算符是当给定的所有值均为非0值，并且都不为NULL时，返回1；当给定的一个值或者多个值为0时则返回0；否则返回NULL。

```SQL
SELECT 1 AND -1, 0 AND 1, 0 AND NULL, 1 AND NULL;
/*输出
+----------+---------+------------+------------+
| 1 AND -1 | 0 AND 1 | 0 AND NULL | 1 AND NULL |
+----------+---------+------------+------------+
|        1 |       0 |          0 |       NULL |
+----------+---------+------------+------------+
*/

SELECT employee_id, last_name, job_id,salary FROM employees WHERE salary >=10000 AND job_id LIKE '%MAN%';
/*输出
+-------------+-----------+--------+----------+
| employee_id | last_name | job_id | salary   |
+-------------+-----------+--------+----------+
|         114 | Raphaely  | PU_MAN | 11000.00 |
|         145 | Russell   | SA_MAN | 14000.00 |
|         146 | Partners  | SA_MAN | 13500.00 |
|         147 | Errazuriz | SA_MAN | 12000.00 |
|         148 | Cambrault | SA_MAN | 11000.00 |
|         149 | Zlotkey   | SA_MAN | 10500.00 |
|         201 | Hartstein | MK_MAN | 13000.00 |
+-------------+-----------+--------+----------+
*/
```

#### 逻辑或运算符

逻辑或（OR或||）运算符是当给定的值都不为NULL，并且任何一个值为非0值时，则返回1，否则返回0；当一个值为NULL，并且另一个值为非0值时，返回1，否则返回NULL；当两个值都为NULL时，返回NULL。

```SQL
SELECT 1 OR -1, 1 OR 0, 1 OR NULL, 0 || NULL, NULL || NULL;
/*输出
+---------+--------+-----------+-----------+--------------+
| 1 OR -1 | 1 OR 0 | 1 OR NULL | 0 || NULL | NULL || NULL |
+---------+--------+-----------+-----------+--------------+
|       1 |      1 |         1 |      NULL |         NULL |
+---------+--------+-----------+-----------+--------------+
*/

#查询基本薪资不在9000-12000之间的员工编号和基本薪资
 SELECT employee_id,salary FROM employees WHERE NOT (salary >= 9000 AND salary <= 12000);
 #或
 SELECT employee_id,salary FROM employees WHERE salary <9000 OR salary > 12000;
 #或
 SELECT employee_id,salary FROM employees WHERE salary NOT BETWEEN 9000 AND 12000;
 /*输出：
+-------------+----------+
| employee_id | salary   |
+-------------+----------+
|         100 | 24000.00 |
|         101 | 17000.00 |
|         102 | 17000.00 |
|         104 |  6000.00 |
|         105 |  4800.00 |
|         106 |  4800.00 |
|         107 |  4200.00 |
*/


SELECT employee_id, last_name, job_id, salary FROM employees WHERE salary >= 10000 OR job_id LIKE '%MAN%';
/*输出
+-------------+-----------+---------+----------+
| employee_id | last_name | job_id  | salary   |
+-------------+-----------+---------+----------+
|         100 | King      | AD_PRES | 24000.00 |
|         101 | Kochhar   | AD_VP   | 17000.00 |
|         102 | De Haan   | AD_VP   | 17000.00 |
|         108 | Greenberg | FI_MGR  | 12000.00 |
*/
```

注意：
 OR可以和AND一起使用，但是在使用时要注意两者的优先级，由于AND的优先级高于OR，因此先对AND两边的操作数进行操作，再与OR中的操作数结合。

#### 逻辑异或运算符

逻辑异或（XOR）运算符是当给定的值中任意一个值为NULL时，则返回NULL；如果两个非NULL的值都是0或者都不等于0时，则返回0；如果一个值为0，另一个值不为0时，则返回1.

```SQL
 SELECT 1 XOR -1, 1 XOR 0,0 XOR 0, 1 XOR NULL, 1 XOR 1 XOR  1, 0  XOR 0 XOR 0;
 /*输出：
+----------+---------+---------+------------+----------------+----------------+
| 1 XOR -1 | 1 XOR 0 | 0 XOR 0 | 1 XOR NULL | 1 XOR 1 XOR  1 | 0  XOR 0 XOR 0 |
+----------+---------+---------+------------+----------------+----------------+
|        0 |       1 |       0 |       NULL |              1 |              0 |
+----------+---------+---------+------------+----------------+----------------+
*/


select last_name,department_id,salary from employees where department_id in (10,20) XOR salary > 8000;
/*输出：
+------------+---------------+----------+
| last_name  | department_id | salary   |
+------------+---------------+----------+
| King       |            90 | 24000.00 |
| Kochhar    |            90 | 17000.00 |
| De Haan    |            90 | 17000.00 |
| Hunold     |            60 |  9000.00 |
| Greenberg  |           100 | 12000.00 |
| Faviet     |           100 |  9000.00 |
*/
```

### 位运算符

> 后面再补充

## 排序与分页

### 排序数据

#### 排序规则

使用 `ORDER BY `子句排序  `ASC`（ascend）: 升序  `DESC`（descend）:降序
 ORDER BY 子句在SELECT语句的结尾。

#### 单列排序

按照雇佣时间升序查找

```SQL
SELECT last_name, job_id, department_id, hire_date
FROM employees
ORDER BY hire_date ;
```

#### 多列排序

可以使用不在SELECT列表中的列排序。

在对多列进行排序的时候，首先排序的第一列必须有相同的列值，才会对第二列进行排序。如果第一列数据中所有值都是唯一的，将不再对第二列进行排序。

```SQL
SELECT last_name, department_id, salary
FROM employees
ORDER BY department_id, salary DESC;
/*部分输出
+-------------+---------------+----------+
| last_name   | department_id | salary   |
+-------------+---------------+----------+
| Grant       |          NULL |  7000.00 |
| Whalen      |            10 |  4400.00 |
| Hartstein   |            20 | 13000.00 |
| Fay         |            20 |  6000.00 |
| Raphaely    |            30 | 11000.00 |
| Khoo        |            30 |  3100.00 |
| Baida       |            30 |  2900.00 |
| Tobias      |            30 |  2800.00 |
*/
```

### 分页数据

所谓分页显示，就是将数据库中的结果集，一段一段显示出来需要的条件。  MySQL中使用 `LIMIT `实现分页

格式：`LIMIT `位置偏移量, 行数
 第一个“位置偏移量”参数指示MySQL从哪一行开始显示，是一个可选参数，如果不指定“位置偏移量”，将会从表中的第一条记录开始（第一条记录的位置偏移量是0，第二条记录的位置偏移量是1，以此类推）；第二个参数“行数”指示返回的记录条数。

分页显式公式：（当前页数-1）*每页条数，每页条数:

```SQL
SELECT * FROM table
LIMIT(PageNo - 1)*PageSize,PageSize;
```

注意：LIMIT 子句必须放在整个SELECT语句的最后！

## 多表查询(重点）

> *多表查询，也称为关联查询，指两个或更多个表一起完成查询操作。前提条件：这些一起查询的表之间是有关系的（一对一、一对多），它们之间一定是有关联字段，这个关联字段可能建立了外键，也可能没有建立外键。比如：员工表和部门表，这两个表依靠“部门编号”进行关联。*

### 内连接

#### 内连接查询语法：

隐式内连接：SELECT 字段列表 FROM 表1， 表2 WHERE 条件...;

显式内连接：SELECT 字段列表 [INNER] JOIN 表2 ON 连接条件...;

内连接查询的是两张表交集的部分

![img](https://secure2.wostatic.cn/static/2yX4ZqR1XazmU4P6cCCDb3/image.png)

查询每一个员工的姓名，及关联的部门名称（隐式内连接实现）

```SQL
SELECT emp.name,dept.name from emp,dept where emp.dept_id = dept.id;
```

显式内连接实现

```SQL
SELECT e.name ,d.name,from emp e inner join dept d on e.dept_id = d.id
```

### 外连接

#### 外连接查询语法

**左外连接**~~:~~相当于查询表1（左表）的所有数据，包含表1和表2交集部分的数据

```SQL
SELECT 字段列表 FROM 表1 LEFT[OUTER] JOIN 表2 ON 条件...;
```

右外连接:相当于查询表2（右表）的所有数据 包含表1和表2交集部分的数据

```SQL
SELECT 字段列表 FROM 表1 RIGHT [OUTER] JOIN 表2 ON 条件...;
```

### 自连接

#### 自连接处查询语法

自连接就是自己查询自己表中信息的时候用。比如

查询员工 及其 所属领导的名字，在员工表中，有员工自己的id，还有领导id(managerid),可以通过自连接查询到。

```SQL
SELECT a.name, b.name FROM emp a, emp b WHERE a.managerid= b.id  
```

查询所有员工 emp 及其领导的名字 如果没有领导 也要查出来

```SQL
SELECT a.name '员工' ,b.name '领导' FROM emp a left join emp b ON a.managerid = b.id 
```

### 联合查询

使用关键字 union 查询，把多次查询的结果合并起来，形成一个新的查询结果集

```SQL
SELECT 字段列表 FROM 表A
UNION[ALL]
SELECT 字段列表 FROM 表B....;
```

注：对于联合查询的多张表的列数必须保持一致，字段类型也需要保持一致。union all会将全部数据直接合并在一起，union会对合并之后的数据去重。

### 子查询

SQL语句中嵌套SELECT 语句，成为嵌套查询，又称子查询

```SQL
SELECT * FROM t1 WHERE column1 =（SELECT column1 FROM t2)
```

子查询外部的语句可以是INSERT/ UPDATE/DELETE/SELECT的任何一个。

根据子查询的结果不同，分为：

- 标量子查询
- 列子查询
- 行子查询
- 表子查询

#### 标量子查询

子查询返回的结果是单个值 常用操作符：- < > > >= < <=

查询销售部所有员工

```SQL
-- 合并（子查询）
select * from employee where dept = (select id from dept where name = '销售部');
```

-- 查询xxx入职之后的员工信息`

```SQL
select * from employee where entrydate > (select entrydate from employee where name = 'xxx');
```

#### 列子查询

返回的结果是一列（可以是多行）

常用操作符：

| 操作符 | 描述                                   |
| ------ | -------------------------------------- |
| IN     | 在指定的集合范围内，多选一             |
| NOT IN | 不在指定的集合范围内                   |
| ANY    | 子查询返回列表中，有任意一个满足即可   |
| SOME   | 与ANY等同，使用SOME的地方都可以使用ANY |
| ALL    | 子查询返回列表的所有值都必须满足       |

查询销售部和市场部的所有员工信息

```SQL
select * from employee where dept in (select id from dept where name = '销售部' or name = '市场部');
```

查询比财务部所有人工资都高的员工信息

```SQL
select * from employee where salary > all(select salary from employee where dept = (select id from dept where name = '财务部'));
```

#### 行子查询

返回的结果是一行（可以是多列）。  常用操作符：=, <, >, IN, NOT IN

查询与xxx的薪资及直属领导相同的员工信息

```SQL
select * from employee where (salary, manager) = (12500, 1);
select * from employee where (salary, manager) = (select salary, manager from employee where name = 'xxx');
```

#### 表子查询

返回的结果是多行多列  常用操作符：IN

查询与xxx1，xxx2的职位和薪资相同的员工

```SQL
select * from employee where (job, salary) in (select job, salary from employee where name = 'xxx1' or name = 'xxx2');
```

查询入职日期是2006-01-01之后的员工，及其部门信息

```SQL
select e.*, d.* from (select * from employee where entrydate > '2006-01-01') as e left join dept as d on e.dept = d.id;
```