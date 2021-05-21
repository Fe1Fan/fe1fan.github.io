---
title: "MySQL 笔记1：基础"
date: 2020-10-07T14:23:52+08:00
tags: [mysql]
math: true
---
<!--more--> 
## 字段数据类型
- 整数
  - TINYINT  (8)
  - SMALLINT  (16)
  - MEDIUMINT  (24)
  - INT  (32)
  - BIGINT  (64)
- 实数
  - FLOAT  (4)
  - DOUBLE  (8)
  - DECIMAL
- 字符串
  - CHAR
  - VARCHAR
  - BLOB
  - TEXT
- 枚举
  - ENUM
- 日期和时间
  - DATETIME
  - TIMESTAMP
- 位数据
  - BIT
  - SET

---
### 整数
$$ 2^(N-1) 到 2^(N-1) -1$$
如果使用UNSIGNED 大致可以使正数的上限提高一倍，例如TINYINT UNSIGNED 可以存储的范围是0～255，而TINYINT的存储范围是−128～127。

有符号和无符号类型使用相同的存储空间，并具有相同的性能。

整数型可指定宽度，但指定宽度不影响存储和计算，只用于交互工具的展示。


[comment]: <> (在MySQL 4.1以后的版本中，InnoDB可以将每个表的数据和索引存放在单独的文件中。)

[comment]: <> (MyISAM会将表存储在两个文件中：数据文件和索引文件，分别以.MYD 和.MYI 为扩展名。)

---
### 实数
DECIMAL 类型都可以指定精度，例如，DECIMAL（18,9） 小数点两边将各存储9个数字，一共使用9个字节：小数点前的数字用4个字节，小数点后的数字用4个字节，小数点本身占1个字节。

因为需要额外的空间和计算开销，所以应该尽量只在对小数进行精确计算时才使用DECIMAL ——例如存储财务数据，

在数据量比较大的时候，可以考虑使用BIGINT 代替DECIMAL ，将需要存储的货币单位根据小数的位数乘以相应的倍数即可。

---
### 字符串

#### CHAR
> CHAR 类型是定长的

当存储CHAR 值时，MySQL会删除所有的末尾空格

CHAR类型适合存储很短的字符串，或者所有值都接近同一个长度。例如MD5值，因为定长，不容易出现碎片化问题，所以在数据更变的时候会比VARCHAR更好。

假如字段 is_student 需要用 Y 或者 N 来表示，CHAR(1)使用一个字节来进行存储，而VARCHAR(1)会使用两个字节存储，多余的一个字节存储的是VARCHAR字节数。


#### VARCHAR
> 用于存储可变长字符串

VARCHAR 需要使用1或2个额外字节记录字符串的长度：
如果列的最大长度小于或等于255字节，则只使用1个字节表示，否则使用2个字节。(粗略概述，详情见[InnoDB存储](https://fe1.fan/posts/mysql/mysql-note-6/?uid=91a329063146eef058647ad7f523b337))

扩展：
如果一个行占用的空间增长，并且在页内没有更多的空间可以存储，在这种情况下，不同的存储引擎的处理方式是不一样的。

- MyISAM

将行拆成不同的片段存储

- InnoDB

需要分裂页来使行可以放进页内。

VARCHAR(5) 和 VARCHAR(200) 在储存的开销上是一样的，但是在加载入内存的时候，会分配固定大小的内存块，所以尽量分配真正使用的空间。

#### BLOB
> 采用二进制形式存储的很大的字符串数据类型

- TINYBLOB 
- SMALLBLOB
- BLOB
- MEDIUMBLOB
- LONGBLOB

当数据过大时，MySQL以单独的对象存储，InnoDB会使用专门的"外部"存储区来存储，此时每个值在行内需要1～4个字节存储一个指针，然后在外部存储区域存储实际的值。

详情见[InnoDB存储](https://fe1.fan/posts/mysql/mysql-note-6/?uid=91a329063146eef058647ad7f523b337)

BLOB没有排序规则和字符集。


#### TEXT

- TINYTEXT 
- SMALLTEXT 
- TEXT
- MEDIUMTEXT 
- LONGTEXT

当数据过大时，MySQL以单独的对象存储，InnoDB会使用专门的"外部"存储区来存储，此时每个值在行内需要1～4个字节存储一个指针，然后在外部存储区域存储实际的值。

详情见[InnoDB存储](https://fe1.fan/posts/mysql/mysql-note-6/?uid=91a329063146eef058647ad7f523b337)

TEXT有排序规则和字符集。

---
### 枚举
在如上所示的 is_student 字段中，可以用枚举进行代替字符串。 MySQL会在内部将每个值在列表中的位置保存为整数，并且在表的.frm文件中保存数字--字符串的映射关系，尽量不要使用 ENUM('1','2','3')
这种类型的枚举，因为会容易造成混乱。另外枚举字段是按照内部存储的整数而不是定义的字符串进行排序的。

### 日期和时间

#### DATETIME

这个类型能保存大范围的值，从1001年到9999年，精度为秒。它把日期和时间封装到格式为YYYYMMDDHHMMSS的整数中，与时区无关。使用8个字节的存储空间。


#### TIMESTAMP
TIMESTAMP 类型保存了从1970年1月1日午夜（格林尼治标准时间）以来的秒数，它和UNIX时间戳相同。TIMESTAMP 只使用4个字节的存储空间，因此它的范围比DATETIME 小得多：只能表示从1970年到2038年。

### 位数据
//TODO
## SQL基础

### 查询

`SELECT`

```shell
# 查询列
SELECT col_name FROM table;
# 别名
SELECT col_name AS name FROM table;
# 查询常数
SELCT 'const' AS c, colname FROM table;
# 去重复行
SELECT DISTNCE col_name FROM table;
# 排序
SELECT col_name FROM table ORDER BY col_name DESC;
## 第二排序 首先按照第一个排序，当第一个值相同时按照第二个
SELECT col_name FROM table ORDER BY col_name_1, col_name_2 DESC;
# 约束返回结果
SELECT col_name FROM table LIMIT 1;
# SELECT 语句顺序：
SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...
# SELECT 执行顺序
FROM > WHERE > GROUP BY > HAVING > SELECT 的字段 > DISTINCT > ORDER BY > LIMIT
```
在 SELECT 语句执行这些步骤的时候，每个步骤都会产生一个虚拟表，然后将这个虚拟表传入下一个步骤中作为输入。需要注意的是，这些步骤隐含在 SQL 的执行过程中，对于我们来说是不可见的。

### 条件

`WHERE`

运算符：

|含义|运算符|
|----|----|
|等于|=|
|不等于|<> 或 !=|
|大于|>|
|小于|<|
|小于等于|<=|
|大于等于|>=|
|两数之间|BETWEEN...AND...|
|为空|IS NULL|
|不为空|IS NOT NULL|

```shell
# >
SELECT col_name FROM table WHERE col_name > 1;
# 两数之间
SELECT col_name FROM table WHERE col_name BETWEEN 1 AND 2;

```

逻辑运算符

|含义|运算符|
|----|----|
|并且|AND|
|或者|OR|
|指定范围|IN|
|否定|NOT|
|通配符|LIKE|

### 常用函数

|函数名|定义|
|----|----|
|ABS()|取绝对值|
|MOD()|取余|
|ROUND()|四舍五入为指定的小数位数，需要两个字参数，分别为字段名称、小数位数|
|CONCAT()|将多个字符串拼接|
|LENGTH()|计算字符长度，一个汉字算三个字符，一个数字或字母算一个字符|
|CHAR_LENGTH()|计算字段的长度，汉字、数字、字母都算一个字符|
|LOWER()|将字符串转换为小写|
|UPPER()|将字符串转换为大写|
|REPLACE()|替换函数，三个参数，分别是要替换的表达式或字段名字，被替换的字符串，要替换成的字符串|
|SUBSTRING()|截取字符串，三个参数，分别是待截取的表达式或字段名，开始截取的位置，想要截取的长度|

//TODO

### 视图

> 相当于是一张表或多张表的数据结果集。

```shell
CREATE VIEW view_name AS
SELECT column1, column2
FROM table
WHERE condition
```

创建之后可以直接使用:

```shell
SELECT * FROM view_name
```

修改视图：

```shell
ALTER VIEW view_name AS
SELECT column1, column2
FROM table
WHERE condition
```

删除视图：

```shell
DROP VIEW view_name
```

### 存储过程

> SQL语句的封装，使用它就像使用函数一样。

创建

```shell
CREATE PROCEDURE 存储过程名称 ([参数列表])
BEGIN
    需要执行的语句
END    
```

example：

```shell
CREATE PROCEDURE `add_num`(IN n INT)
BEGIN
       DECLARE i INT;
       DECLARE sum INT;
       
       SET i = 1;
       SET sum = 0;
       WHILE i <= n DO
              SET sum = sum + i;
              SET i = i +1;
       END WHILE;
       SELECT sum;
END
```
//TODO
