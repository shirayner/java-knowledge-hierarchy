[TOC]





# 前言



# 命令清单

```mysql
-- Database-Level
create database [if not exists] db_name;   -- Create a new database
drop database [if exists] db_name;         -- Delete the database (irrecoverable!)


show databases;                            -- Show all the databases in this server
use  db_name;                              -- Set the default (current) database
select database();                         -- Show the default database
show create database db_name;              -- Show the CREATE DATABASE statement


-- Table Level
CREATE TABLE [IF NOT EXISTS] tbl_name(
  col_name data_type [NOT NULL | NULL] [DEFAULT {literal | (expr)} ]     
  [AUTO_INCREMENT] [UNIQUE [KEY]] [[PRIMARY] KEY] 
) ;

DROP TABLE IF EXISTS  tbl_name...       -- drop table
SHOW CREATE TABLE t \G                  -- Show the create table statement


-- Row-Level
INSERT INTO tableName 
   VALUES (column1Value, column2Value,...)               -- Insert on all Columns
INSERT INTO tableName 
   VALUES (column1Value, column2Value,...), ...          -- Insert multiple rows
   
INSERT INTO tableName (column1Name, ..., columnNName)
   VALUES (column1Value, ..., columnNValue)              -- Insert on selected Columns

DELETE FROM tableName WHERE criteria

UPDATE tableName SET columnName = expr, ... WHERE criteria

SELECT * | column1Name AS alias1, ..., columnNName AS aliasN 
   FROM tableName
   WHERE criteria
   GROUP BY columnName
   ORDER BY columnName ASC|DESC, ...
   HAVING groupConstraints
   LIMIT count | offset count
   
```



# 一、数据库的基本操作

## 1.选定

查看数据库列表：

```mysql
show databases;
```



USE语句选定一个数据库并把它当做指定MySQL服务器连接上的默认（当前）数据库：

```mysql
use db_name
```

选定了一个数据库后，你就可以操作其中的数据表了。



同时可以通过如下语句查看当前连接的默认数据库：

```mysql
select database();
```



## 2.创建

```mysql
CREATE  DATABASE  [IF NOT EXISTS] db_name
    [create_specification] ...

create_specification:
    [DEFAULT] CHARACTER SET [=] charset_name
  | [DEFAULT] COLLATE [=] collation_name
```



在默认情况下，服务器级别的字符集和排序方式将成为新建数据库的默认字符集和排序方式。可以使用CHARACTERSET和COLLATE子句对这些数据库属性作出明确的设置。如下所示：

```mysql
create database mydb character set utf8 collate utf8_icelandic_ci;
```



查看数据库定义语句：

```mysql
mysql> show create database mysql \g
+----------+----------------------------------------------------------------+
| Database | Create Database                                                |
+----------+----------------------------------------------------------------+
| mysql    | CREATE DATABASE `mysql` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+----------------------------------------------------------------+
1 row in set (0.00 sec)

```





## 3.删除

```mysql
DROP DATABASE  [IF EXISTS] db_name
```



## 4.修改

主要是修改数据库的默认字符集和排序方式

```mysql
ALTER DATABASE [db_name]
    alter_specification ...

alter_specification:
    [DEFAULT] CHARACTER SET [=] charset_name
  | [DEFAULT] COLLATE [=] collation_name
```





# 二、数据表的基本操作

## 1.存储引擎

使用如下命令可以查看有哪些存储引擎可供选用：

```mysql
mysql> show engines \G
*************************** 1. row ***************************
      Engine: FEDERATED
     Support: NO
     Comment: Federated MySQL storage engine
Transactions: NULL
          XA: NULL
  Savepoints: NULL
*************************** 2. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 3. row ***************************
      Engine: MyISAM
     Support: YES
     Comment: MyISAM storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 4. row ***************************
      Engine: BLACKHOLE
     Support: YES
     Comment: /dev/null storage engine (anything you write to it disappears)
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 5. row ***************************
      Engine: CSV
     Support: YES
     Comment: CSV storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 6. row ***************************
      Engine: MEMORY
     Support: YES
     Comment: Hash based, stored in memory, useful for temporary tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 7. row ***************************
      Engine: ARCHIVE
     Support: YES
     Comment: Archive storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 8. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
*************************** 9. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
9 rows in set (0.00 sec)

```

可以看到 InnoDB 是默认的引擎。

其中上述存储引擎信息是存储在 information_schema.engines 数据表中的。

```mysql
show databases;
use information_schema;
show tables;
select * from engines;
```



## 2.创建数据表

完整的建表语句可选项很多，请见官方文档：[13.1.20 CREATE TABLE Syntax](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)

### 2.1 基本建表语句

```mysql
CREATE TABLE [IF NOT EXISTS] tbl_name(
  col_name data_type [NOT NULL | NULL] [DEFAULT {literal | (expr)} ]     
  [AUTO_INCREMENT] [UNIQUE [KEY]] [[PRIMARY] KEY] 
) ;
```



示例：

```mysql
CREATE TABLE IF NOT EXISTS tasks (
  task_id INT(11) NOT NULL AUTO_INCREMENT,  
  subject VARCHAR(45) DEFAULT NULL,
  start_date DATE DEFAULT NULL,
  end_date DATE DEFAULT NULL,
  description VARCHAR(200) DEFAULT NULL,
  PRIMARY KEY (task_id)
) ENGINE=InnoDB;


CREATE TABLE
IF
	NOT EXISTS USER (
	id BIGINT ( 12 ) NOT NULL AUTO_INCREMENT,
	user_name VARCHAR ( 45 ) NOT NULL,
	PASSWORD VARCHAR ( 45 ) NOT NULL,
	age INT ( 4 ) UNSIGNED DEFAULT NULL,
	city VARCHAR ( 45 ) DEFAULT NULL,
	start_date DATE DEFAULT NULL,
	end_date DATE DEFAULT NULL,
	description VARCHAR ( 200 ) DEFAULT NULL,
	PRIMARY KEY ( id ) 
	)
```



### 2.2 复制表

#### 2.2.1 like

创建原表的一个空白副本，新表与原表数据结构完全相同。

包括：

- 字段信息、默认值
- 约束、索引等

```mysql
CREATE TABLE B LIKE A
```



若要为数据表创建一份空白副本，再从原始数据表填充它.

```
create table B like A;
insert into B select * from A;
```



#### 2.2.2 select

```
CREATE TABLE B [AS] SELECT * FROM A;
```

上述语句会从一个子查询来创建数据表，创建的表的字段结构。

通过此种方式创建的数据表，可以保留下来的属性包括数据列是NULL还是NOTNULL、字符集和排序方式、默认值和数据列注释，而主键、索引、约束等都是不会复制过来的。



## 3.删除表

```mysql
mysql> DROP TABLE IF EXISTS t1, t2;
```



## 4.修改表

```mysql
-- Add a new column to payments table
ALTER TABLE `payments` ADD COLUMN `staff_id`  INT UNSIGNED  NOT NULL;

-- add an index on column d and a UNIQUE index on column a:
ALTER TABLE t2 ADD INDEX (d), ADD UNIQUE (a);
```



## 5.查看建表语句

```mysql
mysql> SHOW CREATE TABLE t \G
*************************** 1. row ***************************
       Table: t
Create Table: CREATE TABLE `t` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `s` char(60) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```





# 三、数据行的基本操作

## 1.新增

```mysql
-- Row-Level
INSERT INTO tableName 
   VALUES (column1Value, column2Value,...)               -- Insert on all Columns
   
INSERT INTO tableName 
   VALUES (column1Value, column2Value,...), ...          -- Insert multiple rows
   
INSERT INTO tableName (column1Name, ..., columnNName)
   VALUES (column1Value, ..., columnNValue)              -- Insert on selected Columns
```



## 2.删除

```mysql
DELETE FROM tableName WHERE criteria
```



## 3.修改

```mysql
UPDATE tableName SET columnName = expr, ... WHERE criteria
```



## 4.查询

查询语句很复杂，这里只列出简单的查询语句

```mysql
select column_list  from table_list  where criteria
```









# 参考资料

1. [Chapter 13 SQL Statement Syntax----dev.mysql.com](https://dev.mysql.com/doc/refman/8.0/en/sql-syntax.html)
2. [MySQL by Examples for Beginners](http://www.ntu.edu.sg/home/ehchua/programming/sql/mysql_beginner.html)