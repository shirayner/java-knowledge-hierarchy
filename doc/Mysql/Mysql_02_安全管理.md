[TOC]



# 前言



# 命令清单



```mysql
-- 用户管理
select user,host,password  from mysql.user;  			  -- 查看用户
create user 'userName'@'host' identified by 'password';   -- 创建用户
create user  userName  identified by 'password'; 		  -- 创建用户：使用默认host(%)创建用户
rename user  oldName to newName;  						  -- 重命名用户
drop   user  userName;  								  -- 删除用户


-- 权限管理
show grants for userName;   					   -- 查看用户权限
grant privilege on database.table to userName;     -- 赋予用户权限
revoke privilege on database.table from userName;  -- 撤销权限
```





# 一、用户管理

用户的身份由 mysql 用户名和 用于连接的主机名一起决定。



## 1.查看用户



```mysql
mysql> select user,host,password from mysql.user;

+------+-----------------------+-------------------------------------------+
| user | host                  | password                                  |
+------+-----------------------+-------------------------------------------+
| root | localhost             | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| root | localhost.localdomain |                                           |
| root | 127.0.0.1             |                                           |
| root | ::1                   |                                           |
| ray  | %                     | *84A30421EF1FC74623DFD7D60D23EFA5FF199BA1 |
+------+-----------------------+-------------------------------------------+
5 rows in set (0.00 sec)
```



user表中host列的值的意义：

| host      | 含义                                                |
| --------- | --------------------------------------------------- |
| %         | 匹配所有主机                                        |
| localhost | localhost不会被解析成IP地址，直接通过UNIXsocket连接 |
| 127.0.0.1 | 会通过TCP/IP协议连接，并且只能在本机访问；          |
| ::1       | ::1就是兼容支持ipv6的，表示同ipv4的127.0.0.1        |

​             

​        

## 2.创建用户

> 默认host为% 



```mysql
create user 'userName'@'host' identified by  'password'; -- 用userName、host、password创建用户  
create user  userName  identified by  'password';  -- 用userName、password创建用户，默认host为% 
```



示例：

```mysql
 -- 用userName、password创建用户，则默认host为% 
mysql> create user  test1  identified by  'test1';
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host,password from mysql.user;
+----------+-----------------------+-------------------------------------------+
| user     | host                  | password                                  |
+----------+-----------------------+-------------------------------------------+
| root     | localhost             | *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B |
| root     | localhost.localdomain |                                           |
| root     | 127.0.0.1             |                                           |
| root     | ::1                   |                                           |
| ray      | %                     | *84A30421EF1FC74623DFD7D60D23EFA5FF199BA1 |
| test1    | %                     | *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19 |
+----------+-----------------------+-------------------------------------------+
6 rows in set (0.00 sec)

```

示例中创建用户的命令等同于：

```mysql
mysql> create user 'test1'@'%' identified by 'test1';  
```



## 3.重命名用户

```mysql
rename user oldName to newName;  	-- 重命名用户
```



## 4.删除用户

```mysql
drop user userName;  -- 删除用户
```



## 5.修改密码

可通过如下三种方式修改用户的密码：

```mysql
-- 1. shell命令行中 使用mysqladmin修改密码
shell> mysqladmin -u root  password "root";

-- 2. Mysql命令行中
mysql> set password for 'root'@'localhost' = password('root');

-- 3. 用UPDATE直接编辑user表
use mysql;
update user set password=password('root') where user='root' and host='localhost';
flush privileges;
```





# 二、权限管理

注意：

> 修改完权限之后需要刷新权限：`FLUSH PRIVILEGES`

## 0.权限相关表

mysql的权限都是记录在相应的数据表中的：

| 权限层级   | 权限表             | 说明                                                         |
| ---------- | ------------------ | ------------------------------------------------------------ |
| 全局层级   | mysql.user         | 全局权限适用于一个给定服务器中的所有数据库。这些权限存储在`mysql.user`表中。`GRANT ALL ON *.*`和 `REVOKE ALL ON *.* `只授予和撤销全局权限 |
| 数据库层级 | mysql.db           | 数据库权限适用于一个给定数据库中的所有目标。这些权限存储在`mysql.db`表中。`GRANT ALL ON db_name.*`和`REVOKE ALL ON db_name.*`只授予和撤销数据库权限 |
| 表层级     | mysql.talbes_priv  | 表权限适用于一个给定表中的所有列。这些权限存储在`mysql.talbes_priv`表中。`GRANT ALL ON db_name.tbl_name`和`REVOKE ALL ON db_name.tbl_name`只授予和撤销表权限 |
| 列层级     | mysql.columns_priv | 列权限适用于一个给定表中的单一列。这些权限存储在`mysql.columns_priv`表中。当使用REVOKE时，您必须指定与被授权列相同的列。 |









## 1.查看用户权限

```mysql
show grants for userName;
```



## 2.赋予用户权限

```mysql
grant privilege on database.table to userName;
grant all on *.* to 'system'@'localhost' identified by 'clsn123' with grant option; 
```

> with grant option 让被授权的用户也能为他人授权

## 3.撤销用户权限

```mysql
revoke privilege on database.table from userName;
```



## 4.权限示例

GRANT和REVOKE可在几个层次上控制访问权限：

> - 整个服务器，使用 `GRANT ALL` 和 `REVOKE ALL`；
> - 整个数据库，使用 `ON database.*`；
> - 特定的表，使用 `ON database.table`；
> - 特定的列；
> - 特定的存储过程。



```mysql
grant all on *.* to dba@'%'  -- 整个服务器
grant select, insert, update, delete  on testdb.* to dba@'%'  -- 整个数据库
grant select, insert, update, delete  on testdb.user to dba@'%'  -- 特定表
```



可以授权的用户权限

```mysql
INSERT,SELECT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, 
PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, 
CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, 
REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER 
ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE
```





# 参考资料

1.  [mysql 用户管理和权限设置](https://www.cnblogs.com/fslnet/p/3143344.html)
2.  [MySQL用户管理及SQL语句详解](https://www.cnblogs.com/clsn/p/8047028.html)
3.  [MySQL用户管理：添加用户、授权、删除用户](https://www.cnblogs.com/chanshuyi/p/mysql_user_mng.html)
4.  [mysql 的权限体系介绍](http://blog.sae.sina.com.cn/archives/1912)