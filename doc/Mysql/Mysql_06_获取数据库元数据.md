[TOC]







# 前言

MySQL提供了好几种办法供我们获取关于数据库和数据库里各种对象（也就是数据库的元数据）的信息：

- 各种SHOW语句
- 从`INFORMATION_SCHEMA`数据库里的数据表
- 命令行程序，如`mysqlshow`或`mysqldump`



# 一、show 语句

## 1.数据库级别

```mysql
-- 1.列出服务器所管理的数据库：
show databases;

-- 2.查看给定数据库的建库语句：
show create database db_name;
```



## 2.数据表级别

```mysql
-- 1.列出默认数据库或给定数据库里的数据表：
show tables;   -- SHOW TABLES 语句的输出报告里不包括TEMPORARY数据表。
show tables from db_name;

-- 2.数据表的建表语句：
show create table tbl_name;

-- 3.查看数据表里的数据列或索引信息
show columns from tbl_name;  -- DESCRIBE tbl_name和EXPLAIN tbl_name语句是SHOW COLUMNS FROM tbl_name语句的同义语句。
show index from tbl_name;
desc tbl_name; -- 即 describe tbl_name

-- 4.查看默认数据库或某给定数据库里的数据表的描述性信息：
show table status;
show table status from db_name;
```



有几种SHOW语句还可以带有一条`LIKE'pattern'`子句，





#  二、从`INFORMATION_SCHEMA`数据库获取元数据

可以把`INFORMATION_SCHEMA`数据库想象成一个虚拟的数据库，这个数据库里的数据表是一些由不同的数据库元数据构成的视图。

如果你想知道`INFORMATION_SCHEMA`数据库都包含哪些数据表，请使用`SHOW TABLES`命令。

```mysql
show tables in information_schema;
```



- `SCHEMATA、TABLES、VIWS、ROUTINES、TRIGGERS、EVENTS、PARTITIONS、COLUMNS`

    > 关于数据库、数据表、视图、存储例程（storedroutine）、触发器、数据库里的事件、数据表分区和数据列的信息。

- `FILES`

    > 关于NDB硬盘数据文件的信息。

- `TABLE_CONSTRAINS、KEY_COLUMN_USAGE`

    > 关于数据表和数据列上的约束条件的信息，如唯一化索引、外键等。

- `STATISTICS`

    > 关于数据表索引特性的信息。

- `REFERENTIAL_CONSTRAINS`

    > 关于外键的信息。

- `CHARACTER_SETS、COLLATIONS、COLLATION_CHARACTER_SET_APPLICABILITY`

    > 关于所支持的字符集、每种字符集的排序方式、每种排序方式与它的字符集的映射关系的信息。

- `ENGINES、PLUGINS`

    > 关于存储引擎和服务器插件的信息。

- `USER_PRIVILEGES、SCHEMA_PRIVILEGES、TABLE_PRIVILEGES、COLUMN_PRIVILEGES`

    > 全局、数据库、数据表和数据列的权限信息，这些信息分别来自mysql数据库







# 参考资料

