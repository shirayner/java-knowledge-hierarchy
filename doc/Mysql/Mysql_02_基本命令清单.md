[TOC]





# 前言



# 一、基本命令清单

## 1.Database Operation

```mysql
-- Database-Level
CREATE DATABASE [IF NOT EXISTS] database_name;   -- Create a new database
DROP DATABASE [IF EXISTS] database_name;         -- Delete the database (irrecoverable!)

SHOW DATABASES                             -- Show all the databases in this server
USE databaseName                           -- Set the default (current) database
SELECT DATABASE()                          -- Show the default database
SHOW CREATE DATABASE databaseName          -- Show the CREATE DATABASE statement
 
```



### 1.1 create

```mysql
CREATE DATABASE IF NOT EXISTS tempdb;
```



### 1.2 drop

```mysql
DROP DATABASE IF EXISTS tempdb;
```











## 2.Table Operation

```mysql
-- Table-Level
CREATE TABLE [IF NOT EXISTS] tableName (
   columnName columnType columnAttribute, ...
   PRIMARY KEY(columnName),
   [UNIQUE] INDEX|KEY indexName (columnName, ...),
   FOREIGN KEY (columnNmae) REFERENCES tableName (columnNmae)
)

DROP TABLE [IF EXISTS] tableName, ...

ALTER TABLE tableName ...  -- Modify a table, e.g., ADD COLUMN and DROP COLUMN
ALTER TABLE tableName ADD columnDefinition
ALTER TABLE tableName DROP columnName
ALTER TABLE tableName ADD FOREIGN KEY (columnNmae) REFERENCES tableName (columnNmae)
ALTER TABLE tableName DROP FOREIGN KEY constraintName

SHOW TABLES                -- Show all the tables in the default database
DESCRIBE|DESC tableName    -- Describe the details for a table

SHOW CREATE TABLE tableName        -- Show the CREATE TABLE statement for this tableName
```



### 1.2 create table

```mysql
-- Create the table "products". Read "explanations" below for the column defintions
mysql> CREATE TABLE tasks (
        task_id INT NOT NULL,
        subject VARCHAR(45) NULL,
        start_date DATE NULL,
        end_date DATE NULL,
        description VARCHAR(200) NULL,
        PRIMARY KEY (task_id),
        UNIQUE INDEX task_id_unique (task_id ASC)
	);
Query OK, 0 rows affected (0.08 sec)
```



### 1.3 drop table

```mysql
mysql> DROP TABLE IF EXISTS t1, t2;
```



### 1.4 alter table

```mysql
-- Add a new column to payments table
ALTER TABLE `payments` ADD COLUMN `staff_id`  INT UNSIGNED  NOT NULL;

-- Need to set to a valid value, before adding the foreign key
UPDATE `payments` SET `staff_id` = 8001;
ALTER TABLE `payments` ADD FOREIGN KEY (`staff_id`) REFERENCES staff (`staff_id`) 
   ON DELETE RESTRICT ON UPDATE CASCADE;
 
SHOW CREATE TABLE `payments` \G
SHOW INDEX FROM `payments` \G
```





## 3.Row Operation

````mysql
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
````







# 参考资料

1. [Chapter 13 SQL Statement Syntax----dev.mysql.com](https://dev.mysql.com/doc/refman/8.0/en/sql-syntax.html)
2. [MySQL by Examples for Beginners](http://www.ntu.edu.sg/home/ehchua/programming/sql/mysql_beginner.html)
3. [Mysql教程——易百教程](https://www.yiibai.com/mysql/create-drop-database.html)







