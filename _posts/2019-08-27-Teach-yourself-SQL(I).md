---
layout: post
title: "SQL 笔记（上集）"
comments: true
date: 2019-08-27
description: ""
keywords: "python, mysql, sql, database, mariadb"
---

> 本文为个人总结的 SQL 笔记（上集），参考自《SQL 必知必会（第 4 版）》，人民邮电出版社出版。

> 本文所使用的 SQL 服务端为 MariaDB。

## MySQL in Python

> Python-MySQL tutorial in `w3schools.com` 
> [https://www.w3schools.com/python/python_mysql_getstarted.asp](https://www.w3schools.com/python/python_mysql_getstarted.asp)

### Install MySQL driver

In the link above.

### Create connection

``` python
import mysql.connector
mydb = mysql.connector.connect(
host="localhost",
user="yourusername",
passwd="yourpassword",
database="mydatabase"
)
```

### Create database

``` python
import mysql.connector

mydb = mysql.connector.connect(
host="localhost",
user="yourusername",
passwd="yourpassword"
)

mycursor = mydb.cursor()

mycursor.execute("CREATE DATABASE mydatabase")
```

### Check if database exists

``` python
import mysql.connector

mydb = mysql.connector.connect(
host="localhost",
user="yourusername",
passwd="yourpassword"
)

mycursor = mydb.cursor()

mycursor.execute("SHOW DATABASES")

for x in mycursor:
    print(x)
```

### Create table

``` python
import mysql.connector

mydb = mysql.connector.connect(
host="localhost",
user="yourusername",
passwd="yourpassword",
database="mydatabase"
)

mycursor = mydb.cursor()

mycursor.execute("CREATE TABLE table1 (name VARCHAR(255), address VARCHAR(255))")
```

### Select

* Using `fetchall()` method

``` python
import mysql.connector

mydb = mysql.connector.connect(
host="localhost",
user="yourusername",
passwd="yourpassword",
database="mydatabase"
)

mycursor = mydb.cursor()

mycursor.execute("SELECT * FROM table1")

myresult = mycursor.fetchall()

for x in myresult:
    print(x)
```

* Using `fetchone()` method

The `fetchone()` method will return the first row of the result

``` python
import mysql.connector

mydb = mysql.connector.connect(
host="localhost",
user="yourusername",
passwd="yourpassword",
database="mydatabase"
)

mycursor = mydb.cursor()

mycursor.execute("SELECT * FROM table1")

myresult = mycursor.fetchone()

print(myresult)
```

### Conclusion

引用

``` python
import mysql.connector
```

连接

``` python
mydb = mysql.connector.connect(
host="localhost",
user="yourusername",
passwd="yourpassword",
database="mydatabase"
)
```

执行 SQL 语句

``` python
mycursor.execute("SQL COMMANDS")
```

结果

``` python
myresult = mycursor.fetchall()

for x in myresult:
    print(x)
```

 ## MySQL Commands
 
以下是个人总结。

### 检索数据

一列或多列

``` sql
SELECT column1, column2, ...
FROM table1;
```

所有列

``` sql
SELECT *
FROM table1;
```

只返回不同的值，作用于 `DISTINCT` 后所有的列

``` sql
SELECT DISTINCT column1
FROM table1;
```

限制前5行结果

``` sql
SELECT column1
FROM table1
LIMIT 5;
```

从第5行开始限制前4行

``` sql
SELECT column1
FROM table1
LIMIT 5 OFFSET 4;
```

上面的简化版，注意数字顺序

``` sql
SELECT column1
FROM table1
LIMIT 4,5;
```

注释

``` 
-- 注释形式1
# 注释形式2（少用）
/*注释形式3*/
```

### 排序检索数据

SQL中的子句， `FROM` 和 `ORDER BY` 都属于子句

按 *column1* 排序

``` sql
SELECT column1
FROM table1
ORDER BY column1;
```

按多个列排序

``` sql
SELECT column1, column2, column3, ...
FROM table1
ORDER BY column1, column2;
```

按列位置排序

``` sql
SELECT column1, column2, column3, ...
FROM table1
ORDER BY 2, 3;
```

指定排序方向，升序（默认） `ASC` ，降序 `DESC` 

``` sql
SELECT column1, column2, column3, ...
FROM table1
ORDER BY column1 DESC;
```

指定多列排序方向
*column1* 降序， *column2* 升序（默认）

``` sql
SELECT column1, column2, column3, ...
FROM table1
ORDER BY column1 DESC, column2;
```

特别地，按字典排序

``` sql
无法用 `ORDER BY` 做到，需要改变数据库的设置方式
```

### 过滤数据

`WHERE` 子句

``` sql
SELECT column1, column2, ...
FROM table1
WHERE column1 = xxx;
```

`WHERE` 子句操作符

``` sql
=, >, <, <>, !=, <=, >=, !<, !>, BETWEEN, IS NULL;
```

检查单个值

``` sql
SELECT column1, column2, ...
FROM table1
WHERE column1 <= 10;
```

不匹配检查

``` sql
SELECT column1, column2, ...
FROM table1
WHERE column1 != 'xxx';
```

范围值检查

``` sql
SELECT column1, column2, ...
FROM table1
WHERE column1 BETWEEN 5 AND 10;
```

空值检查

``` sql
SELECT column1, column2, ...
FROM table1
WHERE column1 IS NULL;
```

### 高级数据过滤（操作符）

组合 `WHERE` 子句
此时如有 `ORDER BY` 子句应放在 `WHERE` 子句后

#### `AND` 操作符

``` sql
SELECT column1, column2, column3, ...
FROM table1
WHERE column1 = 'xxx' AND column2 <= 5;
```

#### `OR` 操作符

``` sql
SELECT column1, column2, column3, ...
FROM table1
WHERE column1 = 'xxx' OR column2 <= 5;
```

`AND` 和 `OR` 的处理顺序

**_例子：_**

**输入**

``` sql
SELECT column1, column2
FROM table1
WHERE column3 = 'xxx' OR column4 = 'yyy' AND column2 >= 10;
```

**输出**

| column1 | column2 |
| --- | --- |
| aaa | 3.49 |
| bbb | 3.49 |
| ccc | 3.49 |
| ddd | 11.99 |
| eee | 4.99 |

**分析**
结果中有4行结果小于10，说明SQL在处理 `OR` 操作符前，优先处理 `AND` 操作符。上例中，SQL理解为：*column4为'yyy'的且column2大于10的所有项，以及column3为'xxx'的所有项。*

**正确的输入**
*加上圆括号*

``` sql
SELECT column1, column2
FROM table1
WHERE (column3 = 'xxx' OR column4 = 'yyy') AND column2 >= 10;
```

**正确输入下的输出**

| column1 | column2 |
| --- | --- |
| ddd | 11.99 |

**对于正确输出的分析**
SQL理解为：*column3为'xxx'或column4为'yyy'的且column2小于10的项。*

#### `IN` 操作符

——相当于 `OR` 操作符，**注意条件要放在圆括号中**

优点：

1. 语法清楚
2. 求值顺序容易管理
3. 比 `OR` 执行得更快
4. 可包含其他 `SELECT` 语句

``` sql
SELECT column1, column2
FROM table1
WHERE column3 IN ('xxx', 'yyy');
```

**以上等价于以下**

``` sql
SELECT column1, column2
FROM table1
WHERE column3 = 'xxx' OR column3 = 'yyy';
```

#### `NOT` 操作符

——否定其后条件，支持用 `NOT` 否定 `IN` 、 `BETWEEN` 、 `EXISTS` 

``` sql
SELECT column1, column2
FROM table1
WHERE NOT column3 = 'xxx';
```

**以上等价于以下**

``` sql
SELECT column1, column2
FROM table1
WHERE column3 <> 'xxx';
```

### 用通配符进行过滤

#### `LIKE` 操作符

``` sql
SELECT column1, column2
FROM table1
WHERE column1 LIKE '%xxx%';
```

**百分号 `%` 通配符**最常使用，表示任何字符出现任意次数（0个、1个、多个）。可在任意位置多个使用。

**注意，搜索区分大小写。**

例外：不能匹配 `NULL` 

``` sql
WHERE column1 LIKE '%';
```

不会匹配 column1 中为 `NULL` 的行。

``` sql
SELECT column1, column2
FROM table1
WHERE column1 LIKE '__ xxx';
```

上面出现了下划线通配符 `_` ，用途与 `%` 一样，但它**只匹配单个字符**。

``` sql
SELECT column1, column2
FROM table1
WHERE column1 LIKE '[xy]%';
```

模式 `[xy]%` 中方括号表示匹配方括号内任意**一个字符**（x或y），此通配符可以用前缀字符 `^` 来表示否定。

``` sql
SELECT column1, column2
FROM table1
WHERE column1 LIKE '[^xy]%';
```

以上表示匹配除 x 或 y 以外的任意项。

#### 使用通配符技巧

* 使用通配符一般比其他搜索耗费更多时间，不要过度使用。
* 尽量不要用在搜索模式的开始处，否则搜索起来是最慢的。
* 仔细注意通配符的位置。

### 创建计算字段

是为了直接从数据库中检索出转换、计算或格式化过的数据。

#### 拼接

将值联结在一起，构成单个值。

使用 SQL Server： `+` 

``` sql
#+号联结
SELECT column1 + 'x' + column2 + 'y'
FROM table1;
```

使用 MySQL 或 MariaDB： `Concat()` ；该值是没有列名的，**需要命名列名使用 `AS` 语句**。

``` sql
#Concat()联结
SELECT Concat(column1, 'x', column2, 'y')
AS new_column_name
FROM table1;
```

#### 执行算术计算

``` sql
#乘法
SELECT column1, column2, 
column3 * column4 AS new_column_name
FROM table1
WHERE column3 = xxx;
```

> `SELECT` 语句为测试、检验函数和计算提供了很好的方法。忽略 `FROM` 子句后就是简单的访问和处理表达式： `SELECT 3 * 2;` 将返回 `6` ； `SELECT Trim('abc');` 将返回 `abc` ； `SELECT Now();` 将返回当前日期和时间。

### 使用函数处理数据

#### 函数的问题

> **注意**：函数表达在不同的 DBMS 中的语法不一样，使用函数应当做好代码注释。

#### 使用函数

大多数 SQL 支持以下函数：

* **处理文本字符串**：删除或填充值、转换大写小写
* **数值数据的算术操作**：返回绝对值、代数运算
* **处理日期和时间值**：返回两个日期之差、检查日期有效性
* **返回 DBMS 正使用的特殊信息**：返回用户登录函数

##### 1. 文本处理

``` sql
#转换大写
SELECT column1, UPPER(column1) AS new_column_name_upcase
FROM table1;
```

常用文本处理函数：

``` sql
LEFT() #返回字符串左边的字符
LENGTH() #返回字符串长度
LOWER() #转换字符串为小写
LTRIM() #去除字符串左边的空格
RIGHT() #返回字符串右边的字符
RTRIM() #去除字符串右边的空格
SOUNDEX() #返回字符串的 SOUNDEX 值
UPPER() #转换字符串为大写
```

> `SOUNDEX` 是一个将任何文本串转换为描述其语音表示的字母数字模式的算法，能对字符串进行发音比较而不是字母比较。可以找出发音相似的字符串。

输入：

``` sql
#SOUNDEX()例子
SELECT column1, column2
FROM table1
WHERE SOUNDEX(column2) = SOUNDEX('Michael Green'); #Michael 和 Michelle 发音相近
```

输出：

| column1 | column2 |
| --- | --- |
| xxx | Michelle Green |

##### 2. 日期和时间处理

``` sql
SELECT column1
FROM table1
WHERE YEAR(column2) = 2012; #MySQL\MariaDB 形式
```

##### 3. 数值处理

常用数值处理函数：

``` sql
ABS() #返回绝对值
COS() #返回余弦值
EXP() #返回指数值
PI() #返回 𝛑 值
SIN() #返回正弦值
SQRT() #返回平方根
TAN() #返回正切值
```

### 汇总数据

#### 聚集数据

``` sql
AVG() #某列均值
COUNT() #某列行数
MAX() #某列最大值，要求指定列名，忽略列值为 NULL 的行；用于文本数据时，返回该列排序后的最后一行
MIN() #某列最小值，要求指定列名，忽略列值为 NULL 的行；用于文本数据时，返回该列排序后的最前一行
SUM() #某列值之和，忽略列值为 NULL 的行
```

用法：

``` sql
SELECT AVG(column1) AS new_column_name_avg
FROM table1
WHERE column2 = 'xxx';
```

``` sql
SELECT COUNT(*) #对表中行数（不论列中值 NULL 或非 NULL ）进行计数
SELECT COUNT(column1) #对特定列中具有值（不包含 NULL 值）的行进行计数
```

``` sql
SELECT SUM(column1 * column2) AS new_column_name_sum #用于合计计算值
FROM table1;
```

#### 聚集不同值

使用 `DISTINCT` ：

``` sql
SELECT AVG(DISTINCT column1) AS new_column_name_avg
FROM table1;
```

`DISTINCT` 只能用于 `COUNT(column1)` ，不能用于 `COUNT(*)` 。并且， `DISTINCT` 必须使用列名。

其他聚合函数： `TOP`  `TOP PERCENT` 

#### 组合聚合函数

``` sql
SELECT COUNT(*) AS new_column_name_count, MIN(column1) AS new_column_name_min,
MAX(column1) AS new_column_name_max,
AVG(column1) AS new_column_name_avg
FROM table1;
```

### 分组数据

#### 创建分组

``` sql
SELECT column1, COUNT(*) AS new_column_name_count
FROM table1
GROUP BY column1
```

**使用 `GROUP BY` 子句前的一些重要规定：**

* 可以包含任意数目的列，因而可以对分组进行嵌套
* 如嵌套了分组，数据将在最后的分组上进行汇总（指定的所有列都在一起计算，不能从个别列的列取数据）
* 每一列都必须是检索列或有效的表达式（不能是聚集函数），如在 `SELECT` 中使用表达式，则必须在 `GROUP BY` 子句中指定相同的表达式，不能使用别名
* 大多数 SQL 不允许 `GROUP BY` 列带有长度可变的数据类型（如文本或备注型字段）
* 除聚集计算语句外， `SELECT` 语句中的每一列都必须在 `GROUP BY` 子句中给出
* 如分组列中包含具有 NULL 值的行，则 NULL 将作为一个分组返回；如列中有多行 NULL，它们将分为一组
* `GROUP BY` 子句必须出现在 `WHERE` 子句之后， `ORDER BY` 子句之前

#### 过滤分组

`WHERE` 子句用于过滤行， `WHERE` 没有分组的概念。 `HAVING` 子句用于过滤分组。

> ** `HAVING` 支持所有 `WHERE` 操作符**

``` sql
SELECT column1, COUNT(*) AS new_column_name
FROM table1
GROUP BY column1
HAVING COUNT(*) >= 2; #过滤 `COUNT(*) >= 2` 的分组
```

`WHERE` 在数据分类前进行过滤， `HAVING` 在数据分组后进行过滤。 `WHERE` 排除的行不包括在分组中。

``` sql
#同时使用 `WHERE` 和 `HAVING` 
SELECT column1, COUNT(*) AS new_column_name
FROM table1
WHERE column2 >= 4 #先过滤出 column2 大于等于 4 的行
GROUP BY column1
HAVING COUNT(*) >= 2; #再过滤计数大于等于 2 的分组
```

#### `SELECT` 子句顺序

| 子句 | 说明 | 是否必须使用 |
| --- | --- | --- |
| SELECT  | 需要的列或表达式 | 是 |
| FROM  | 需要的表 | 仅在从表选择数据时使用 |
| WHERE | 行级过滤 | 否 |
| GROUP BY | 分组说明 | 仅在按组计算聚集时使用 |
| HAVING | 组级过滤 | 否 |
| ORDER BY | 排序 | 否 |

### 使用子查询

#### 利用子查询进行过滤

**例：**

第一步：

``` sql
SELECT column1
FROM table1
WHERE column2 = 'xxx';
```

第二步：

``` sql
SELECT column3
FROM table2
WHERE column1 IN (xxx, yyy); #此为第一步中得到的结果
```

一次查询结合以上两步：

``` sql
SELECT column3
FROM table2
WHERE column1 IN (SELECT column1
                  FROM table1
                  WHERE column2 = 'xxx');
```

**注意：作为子查询的 `SELECT` 语句只能查询单个列，多个列将返回错误。**

#### 作为计算字段使用子查询

``` sql
SELECT column1, column2, 
       (SELECT COUNT(*)
        FROM table1
        WHERE table1.column3 = table2.column3) AS new_column_name #该子查询用于统计 `table1.column3` 与 `table2.column3` 中相同的项数；子句中使用了完全限定列名 `table1.column3` 和 `table2.column3` ，以免两个 `column3` 混淆
FROM table2;
```

