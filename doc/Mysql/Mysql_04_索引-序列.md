[TOC]



# 前言



# 一、索引

来加快查询的技术很多，其中最重要的是索引。通常索引能够快速提高查询速度



## 1.索引的存储分类

索引是在MYSQL的存储引擎层中实现的，而不是在服务层实现的。所以每种存储引擎的索引都不一定完全相同，也不是所有的存储引擎都支持所有的索引类型。

MYSQL目前提供了一下4种索引：

- B-Tree 索引：最常见的索引类型，大部分引擎都支持B树索引。
    - 普通索引：允许索引值重复
    - 唯一索引：不允许索引值重复
    - 主键索引：主键是一种唯一性索引，但它必须指定为“PRIMARY KEY”。
- HASH 索引：这是MEMORY数据表的默认索引类型，但你可以改用BTREE索引来代替这个默认索引。
- R-Tree 索引(空间索引)：空间索引是MyISAM的一种特殊索引类型，主要用于地理空间数据类型。
- Full-text (全文索引)：全文索引也是MyISAM的一种特殊索引类型，主要用于全文索引，InnoDB从MYSQL5.6版本提供对全文索引的支持。





## 2.创建索引

### 2.1 Create Table

可以在创建表的时候直接指定索引

```mysql
CREATE TABLE tbl_name(  
   col_name column_definition 
   PRIMARY KEY(index_column,...);                -- 指定主键索引
   INDEX indexName (index_column,...)      		 -- 指定普通索引
   UNIQUE  indexName (index_column,...)   	 	 -- 指定唯一索引
   FULLTEXT  indexName (index_column,...)   	 -- 指定全文索引
); 
```





例如：

```mysql
CREATE TABLE user(  
   id INT NOT NULL,   
   username VARCHAR(16) NOT NULL,  
   email VARCHAR(16) NOT NULL,  
   PRIMARY KEY(id),
   UNIQUE username_index (username)  
); 
```

也可写成

```mysql
CREATE TABLE user(  
   id INT NOT NULL PRIMARY KEY,   
   username VARCHAR(16) NOT NULL UNIQUE,  
   email VARCHAR(16) NOT NULL
); 
```





### 2.2 Create Index

```mysql
CREATE INDEX index_name ON tbl_name (index_column,...);               -- 创建普通索引
CREATE UNIQUE INDEX index_name   ON tbl_name (index_column,...);      -- 创建唯一索引
CREATE FULLTEXT INDEX index_name ON tbl_name (index_column,...);      -- 创建全文索引
CREATE SPATIAL INDEX index_name   ON tbl_name (index_column,...);     -- 创建唯一索引
```



### 2.2  Alter Table

```mysql
ALTER TABLE tbl_name ADD PRIMARY KEY (index_column,...);         -- 添加主键索引
ALTER TABLE tbl_name ADD INDEX  index_name (index_column,...);   -- 添加普通索引
ALTER TABLE tbl_name ADD UNIQUE index_name (index_column,...);   -- 添加唯一索引
ALTER TABLE tbl_name ADD FULLTEXT index_name (index_column,...); -- 添加全文索引
ALTER TABLE tbl_name ADD SPATIAL index_name (index_column,...);  -- 添加SPATIAL索引
```



也可以用同一条Alter添加多条索引

```mysql
ALTER TABLE tbl_name ADD PRIMARY KEY (index_column,...), ADD INDEX  index_name (index_column,...), ADD UNIQUE index_name (index_column,...);
```



## 3.删除索引

### 3.1 Drop Index

```mysql
DROP INDEX index_name ON talbe_name
```



### 3.2 Alter Table

```mysql
ALTER TABLE tbl_name DROP INDEX index_name;
ALTER TABLE table_name DROP PRIMARY KEY;
```





## 4.查看索引

```
show index from tbl_name;
```







# 二、序列







# 参考资料

1. [MYSQL-索引](https://segmentfault.com/a/1190000003072424)



