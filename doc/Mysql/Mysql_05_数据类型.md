[TOC]







# 前言

MySQL支持多种数据类型，主要有数值类型、日期/时间类型、字符串类型和二进制类型：

- 数值类型：
    - 整数类型：TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT
    - 浮点小数类型：FLOAT和DOUBLE
    - 定点小数类型：DECIMAL
- 日期/时间类型：包括YEAR、TIME、DATE、DATETIME和TIMESTAMP。
- 字符串类型：
    - 文本字符串：包括CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET等。
    - 二进制字符串：包括BIT、BINARY、VARBINARY、TINYBLOB、BLOB、MEDIUMBLOB和LONGBLOB。





# 一、数值类型

MySQL提供多种数值数据类型，不同的数据类型提供的取值范围不同，可以存储的值的范围越大，其所需要的存储空间也就越大，因此要根据实际需求选择适合的数据类型。

## 1.整数类型

MySQL主要提供的整数类型有：TINYINT、SMALLINT、MEDIUMINT、INT（INTEGER）、BIGINT。

整数类型的字段属性可以添加 `AUTO_INCREMENT` 自增约束条件。



### 1.1 内存大小与取值范围



[**Table 11.1 Required Storage and Range for Integer Types Supported by MySQL**](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

| Type        | Storage (Bytes) | Minimum Value Signed | Minimum Value Unsigned | Maximum Value Signed | Maximum Value Unsigned |
| ----------- | --------------- | -------------------- | ---------------------- | -------------------- | ---------------------- |
| `TINYINT`   | 1               | -128                 | 0                      | 127                  | 255                    |
| `SMALLINT`  | 2               | -32768               | 0                      | 32767                | 65535                  |
| `MEDIUMINT` | 3               | -8388608             | 0                      | 8388607              | 16777215               |
| `INT`       | 4               | -2147483648          | `0`                    | `2147483647`         | `4294967295`           |
| `BIGINT`    | 8               | -2^63^               | `0`                    | 2^63^-1              | 2^64^-1                |



### 1.2 显示宽度

通过整数类型关键字后面的括号内的整数值，可以指定该字段的最大显示宽度，如 int(8) 表示最大有效显示宽度为8。

显示宽度有两个作用：

（1）对齐填充

当整型字段的值的实际个数小于指定显示宽度时，会由空格从左侧填满宽度。

（2）限制位数

只能插入位数小于显示宽度的值



例如：

```mysql
mysql> create table t1(a TINYINT, b SMALLINT, c MEDIUMINT, d INT, e BIGINT, f TINYINT unsigned);  -- 建表
Query OK, 0 rows affected (0.30 sec)

mysql> desc t1;    -- 查看默认显示宽度
+-------+---------------------+------+-----+---------+-------+
| Field | Type                | Null | Key | Default | Extra |
+-------+---------------------+------+-----+---------+-------+
| a     | tinyint(4)          | YES  |     | NULL    |       |
| b     | smallint(6)         | YES  |     | NULL    |       |
| c     | mediumint(9)        | YES  |     | NULL    |       |
| d     | int(11)             | YES  |     | NULL    |       |
| e     | bigint(20)          | YES  |     | NULL    |       |
| f     | tinyint(3) unsigned | YES  |     | NULL    |       |
+-------+---------------------+------+-----+---------+-------+
6 rows in set (0.00 sec)

mysql> insert into t1(a) values(1234); -- 有符号，长度等于显示宽度，无法插入
ERROR 1264 (22003): Out of range value for column 'a' at row 1
mysql> insert into t1(a) values(123);  -- 有符号，长度小于显示宽度，可插入
Query OK, 1 row affected (0.01 sec)

mysql> insert into t1(f) values(1234); -- 无符号，长度等于显示宽度，无法插入
ERROR 1264 (22003): Out of range value for column 'f' at row 1
```





## 2.浮点数类型和定点数类型

MySQL中使用浮点数和定点数表示小数。

浮点类型有两种：单精度浮点类型（FLOAT）和双精度浮点类型（DOUBLE）。定点类型只有一种：DECIMAL。

浮点类型和定点类型都可以用（M，D）来表示，其中M称为精度，表示总共的位数；D称为标度，表示小数的位数。



| **MySQL数据类型** | 存储需求  | **含义**                       |
| ----------------- | --------- | ------------------------------ |
| float(m,d)        | 4字节     | 单精度浮点型  m总个数，d小数位 |
| double(m,d)       | 8字节     | 双精度浮点型  m总个数，d小数位 |
| DECIMAL(M,D)      | M+2个字节 | 压缩的“严格”定点数             |



注意：

（1）不论是定点类型还是浮点类型，如果用户指定的精度值超过精度范围，则会进行四舍五入的处理。

（2）在MySQL中，在对精度要求比较高的时候（如货币、科学数据等），尽量选择使用DECIMAL类型。另外，两个浮点数在进行减法和比较运算的时候容易出问题，因此在使用浮点数类型时需要注意，并尽量避免做浮点数比较。

（3）DECIMAL 类型的数据当插入的小数部分有多余的位数，截断后插入数据库；当插入的整数部分的值超过了其表示范围则不会插入到数据库。

示例：

```mysql
create table t2(a FLOAT(4,1), b DOUBLE(4,1), c DECIMAL(4,1));
insert into t2 values(4.23,4.26,4.234);
```



# 二、日期和时间类型

每个时间类型有一个有效值范围和一个"零"值，当指定不合法的MySQL不能表示的值时使用"零"值。

| 类型      | 大小 (字节) | 范围                                    | 格式                | 用途                     |
| --------- | ----------- | --------------------------------------- | ------------------- | ------------------------ |
| DATE      | 3           | 1000-01-01~9999-12-31                   | YYYY-MM-DD          | 日期值                   |
| TIME      | 3           | '-838:59:59'~'838:59:59'                | HH:MM:SS            | 时间值或持续时间         |
| YEAR      | 1           | 1901~2155                               | YYYY                | 年份值                   |
| DATETIME  | 8           | 1000-01-01 00:00:00~9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| TIMESTAMP | 8           | 1970-01-01 00:00:00~2037 年某时         | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |

示例：

```mysql
create table t3( a YEAR);
insert into t3 values( 2015),(' 2015');
mysql> select * from t3;
+------+
| a    |
+------+
| 2015 |
| 2015 |
+------+
2 rows in set (0.00 sec)

mysql> insert into t3 values('2288');
ERROR 1264 (22003): Out of range value for column 'a' at row 1
```

由上述结果可看出：

> - 插入值无论是数值型还是字符串型，都可以被正确地存储到数据库中；
> - 但是假如插入值超过了YEAR类型的取值范围，则插入失败。





TIMESTAMP 和 DATATIME 除了存储字节和支持的范围不同外，还有一个最大的区别就是：

> - DATETIME 在存储日期数据时，按实际输入的格式存储，即输入什么就存储什么，与时区无关；
>
> - 而 TIMESTAMP 值的存储是以 UTC （世界标准时间）格式保存的，存储时对当前时区进行转换，检索时再转换回当前时区。即查询时，根据当前时区的不同，显示的时间值是不同的。



# 三、字符串类型

## 1.文本字符串



| 类型       | 大小                                          | 用途                                                      |
| ---------- | --------------------------------------------- | --------------------------------------------------------- |
| CHAR       | 0-255字节                                     | 定长字符串                                                |
| VARCHAR    | 0-65535 字节                                  | 变长字符串                                                |
| TINYTEXT   | 0-255字节                                     | 短文本字符串                                              |
| TEXT       | 0-65 535字节                                  | 长文本数据                                                |
| MEDIUMTEXT | 0-16 777 215字节                              | 中等长度文本数据                                          |
| LONGTEXT   | 0-4 294 967 295字节                           | 极大文本数据                                              |
| ENUM       | 枚举类型，只能存一个枚举字符串值              | 1或2个字节，取决于枚举值的数目（最大值65535）             |
| SET        | 一个设置，字符串对象可以有零个或多个 SET 成员 | 1、2、3、4或8个字节，取决于集合成员的数量（最多64个成员） |



### 1.1 CHAR 和 VARCHAR 类型

- CHAR(M) 为固定长度字符串，VARCHAR(M) 是长度可变的字符串.
- CHAR 是固定长度，所以它的处理速度比 VARCHAR 的速度快，但是确定是浪费空间。
- 当检索 CHAR 值时，尾部的空格将被删除掉。VARCHAR 在值保存和检索时尾部的空格仍保留。
- **字符串类型的 M 值是存储的最大字节数，不是显示宽度，如果插入的字符超过了 M 值，则不允许插入。不要与整数类型的 M 值搞混。**



### 1.2 TEXT 类型

TEXT 类型保存非二进制字符串，如文章内容、评论等，当保存或查询 TEXT 列的值时，不删除尾部空格。



### 1.3 ENUM 类型和 SET 类型

ENUM 与SET 都是枚举类型，不同的是，ENUM 字段只能从定义的列值中选择一个值插入，而 SET 类型的列可从定义的列值中选择多个字符的联合。



## 2.二进制字符串

| 类型名称      | 说明                 | 存储需求          |
| ------------- | -------------------- | ----------------- |
| BIT(M)        | 位字段类型           | 大约(M+7)/8个字节 |
| BINARY(M)     | 固定长度二进制字符串 | M个字节           |
| VARBINARY(M)  | 可变长度二进制字符串 | M+1个字节         |
| TINYBLOB(M)   | 非常小的BLOB         | L+1个字节，L<2^8  |
| BLOB(M)       | 小的BLOB             | L+2个字节，L<2^16 |
| MEDIUMBLOB(M) | 中等大小的BLOB       | L+3个字节，L<2^24 |
| LONGBLOB(M)   | 非常大的的BLOB       | L+4个字节，L<2^32 |

BLOB 是二进制字符串，TEXT 是非二进制字符串，两者均可存放大容量的信息。

BLOB 主要存储图片、音频信息等，而 TEXT 只能存储纯文本文件。

但由于现在图片和音频越来越多，检索起来也不方便，所以都不放在数据库，一般放在专门的文件存储服务器上。





# 参考资料

1. [MySQL 数据类型--w3cschool](https://www.w3cschool.cn/mysql/mysql-data-types.html)
2. 





