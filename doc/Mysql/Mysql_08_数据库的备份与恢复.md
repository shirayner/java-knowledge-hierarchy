[TOC]





# 前言



# 一、数据备份的意义

只要发生数据传输、数据存储和数据交换，就有可能产生数据故障。

> - 通常，数据故障可划分为**系统故障**、**事务故障**和**介质故障**三大类。从信息安全的角度出发，实际上第三方或敌方的“信息攻击”，也会产生不同种类的数据故障。例如：计算机病毒型、特洛伊木马型、“黑客”入侵型、逻辑炸弹型等。
> - 这些故障将会造成的**后果**有：**数据丢失**、**数据被修改**、**增加无用数据**及**系统瘫痪**等。
> - 作为系统管理员，就是要千方百计地维护系统和数据的完整性与准确性。通常**采取的措施**有：**安装防火墙**，防止“黑客”入侵；**安装防病毒软件**，采取存取控制措施；**选用高可靠性的软件产品**；增强计算机网络的安全性。
>
> 但是，世界上没有万无一失的信息安全措施。信息世界“攻击和反攻击”也永无止境。对信息的攻击和防护好似矛与盾的关系，螺旋式地向前发展。
>
> 在信息的收集、处理、存储、传输和分发中经常会存在一些新的问题，其中最值得我们关注的就是系统失效、数据丢失或遭到破坏。威胁数据的安全、造成系统失效的主要原因有以下几个方面：硬盘驱动器损坏、人为错误、“黑客”攻击、病毒、自然灾害、电源浪涌、磁干扰等。

因此，数据备份是保护数据的最后手段，也是防止主动型信息攻击的最后一道防线。



## 1.数据备份的意义

数据备份的任务与意义就在于，**当灾难发生后，通过备份的数据完整、快速、简捷、可靠地恢复原有系统**。



## 2.备份时考虑因素

备份时可考虑以下因素：

> - **定期备份**：数据库要定期做备份，备份的周期应当根据应用数据系统可承受的恢复时间，而且定期备份的时间应当在系统负载最低的时候进行。对于重要的数据，要保证在极端情况下的损失都可以正常恢复。
> - **恢复测试**：定期备份后，同样需要定期做恢复测试，了解备份的正确可靠性，确保备份是有意义的、可恢复的。
> - **增量备份**：根据系统需要来确定是否采用增量备份，增量备份只需要备份每天的增量数据，备份花费的时间少，对系统负载的压力也小。缺点就是恢复的时候需要加载之前所有的备份数据，恢复时间较长。
> - `log-bin`：确保MySQL打开了`log-bin`选项，MySQL在做完整恢复或者基于时间点恢复的时候都需要BINLOG。
> - **异地备份**：可以考虑异地备份。





# 二、逻辑备份和恢复

## 1.逻辑备份

逻辑备份也可称为文件级备份，是将数据库中的数据备份为一个文本文件，而备份的大小取决于文件大小。并且该文本文件是可以移植到其他机器上的，甚至是不同硬件结构的机器。



### 1.1 生成`insert`语句备份

> 使用 `mysqldump` 命令生成`insert`语句备份。



**语法**：

```bash
# 备份所有数据库  
shell> mysqldump [options] --all-database  > file_name.sql

# 备份指定的一个或多个数据库  
shell> mysqldump [options] --database DB1 [DB2,DB3...]  > file_name.sql

# 备份指定的数据库或者数据库中的某些表  
shell> mysqldump [options] db_name [tables]  > file_name.sql
```



**示例**：

（1）**备份所有数据库**

```shell
shell>  mysqldump -uroot -p --all-database > /tmp/ dumpback/ alldb.sql
```

（2）**备份指定数据库**

```shell
[root@ gc ～]  mysqldump -uroot -p --database sqoop hive > /tmp/dumpback/sqoop_hive.sql
```

（3）**备份某数据库中的表**

```shell
[root@ gc ～]  mysqldump -uroot -p sqoop tb1 > /tmp/ dumpback/ sqoop_tb1. sql
```



（4）**查看备份内容**

```shell
[root@ gc dumpback]  more  sqoop_tb1.sql
```





**备份的时候需要保证数据一致性**：

（1）**同一时刻取出所有数据**

> 对于事务支持的存储引擎，如Innodb或者BDB等，可以通过控制将整个备份过程在同一个事务中，使用“--single-transaction”选项。示例语句如下:

```shell
[root@ gc ～]  mysqldump --single-transaction test > test_backup.sql
```



（2）**数据库中的数据处于静止状态**

通过锁表参数来完成:

> - LOCK-TABLES 每次锁定一个数据库的表，此参数默认为true(见上面备份内容实例)。
> - LOCK-ALL-TABLES一次锁定所有的表，适用于dump的表分别处于各个不同的数据库中的情况。



### 1.2 生成特定格式的纯文本文件备份

#### 1.2.1 通过`SELECT ... INTO OUTFILE`命令

通过Query将特定数据以指定方式输出到文本文件中，类似于Oracle中的SPOOL功能

参数说明如下:

> - `FIELDS ESCAPED BY ['name']`：在SQL语句中需要转义的字符；
>
> - `FIELDS TERMINATED BY`：设定每两个字段之间的分隔符；
>
> - `FIELDS [OPTIONALLY] ENCLOSED BY 'name'` ：包装，有OPTIONALLY数字类型不被包装，否则全包装；
> - `LINES TERMINATED BY  'name'`：行分隔符，即每记录结束时添加的字符



示例：

```mysql
mysql > select * into outfile '/ tmp/ tb1. txt' 
-> fields terminated by ',' 
-> optionally enclosed by '"' 
-> lines terminated by '\ n' --默 认 
-> from tb1 limit 50;
Query OK, 50 rows affected (0.00 sec) 

[root@ gc tmp] more tb1. txt 
"information_schema"," CHARACTER_SETS"," SYSTEM VIEW" "information_schema"," COLLATIONS"," SYSTEM VIEW" ......
```



#### 1.2.2 通过 `mysqldump`  工具命令导出文本

用此方法可以生成一个文本数据和一个对应的数据库结构创建脚本，主要重要参数如下:



## 2. 逻辑备份的恢复

### 2.1 `INSERT`语句文件的恢复

有如下两种方式：

#### 2.1.1 mysql命令恢复

把sqoop库的tb1表恢复到test库，语句如下：

```bash
shell> mysql -uroot -p -D test < /tmp/dumpback/sqoop_tb1.sql
```



#### 2.1.2 source命令恢复

```shell
shell>  mysql -uroot -proot -D test 
mysql>  select database();
+------------+ 
| database() | 
+------------+ 
| test		 | 
+------------+ 
1 row in set (0.00 sec) 
mysql> source  /tmp/dumpback/sqoop_tb1.sql 
Query OK, 0 rows affected (0.00 sec) Query OK, 0 rows affected (0.00 sec)

```



### 2.2 纯文本文件的恢复

#### 2.2.1 使用`LOAD DATA INFILE `命令

`LOAD DATA INFILE `语句用于高速地从一个文本文件中读取行，并写入一个表中，文件名称必须为一个文字字符串。

此命令是 `SELECT ... INTO OUTFILE` 反操作，

语法：

```mysql
LOAD DATA
    [LOW_PRIORITY | CONCURRENT] [LOCAL]
    INFILE 'file_name'
    [REPLACE | IGNORE]
    INTO TABLE tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [CHARACTER SET charset_name]
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]
    [IGNORE number {LINES | ROWS}]
    [(col_name_or_user_var
        [, col_name_or_user_var] ...)]
    [SET col_name={expr | DEFAULT},
        [, col_name={expr | DEFAULT}] ...]
```



标准示例：

```mysql
LOAD DATA LOCAL INFILE 'data.txt' INTO TABLE tbl_name 
FIELDS TERMINATED BY ',' 
OPTIONALLY ENCLOSED BY '"' 
LINES TERMINATED BY '\n'
```



#### 2.2.2 使用 mysqlimport 导入数据

此工具可用于恢复上面`mysqldump` 生成的`.txt`和`.sql`两文件，所以要保证 `.txt` 文件对应的数据库中的表存在。



```bash
# 首先恢复表结构
shell>  mysql -uroot -p -D test </tmp/tb1.sql

# 恢复数据
shell>  mysqlimport -uroot -p test --fields-enclosed-by =\" --fields-terminated-by =, /tmp/tb1. txt

```







# 三、物理备份和恢复

物理备份和逻辑备份相比最大优点是备份和恢复的速度都很快，这是因为物理备份的原理都是基于文件拷贝。

物理备份分为如下两种：

（1）冷备份：最直接的备份方式

> - 首先停掉数据库服务，然后CP数据文件，恢复时停止MySQL，先进行操作系统级别恢复数据文件，然后重启MySQL服务，使用MYSQLBINLOG工具恢复自备份以来的所有BINLOG。
>
> 这种方式虽然简单，而且对各个存储引擎都支持，但是有一个非常大的弊端就是需要关闭数据库服务。在当前的大多信息系统都是不允许长时间停机的。



（2）热备份：对于不同的存储引擎方法也不同

①MyISAM存储引擎，本质就是将要备份的表加读锁，然后再CP数据文件到备份目录。方法有如下两种。

方法1：使用MYSQLHOTCOPY工具。MYSQLHOTCOPY工具是MySQL自带的热备份工具，

```mysql
mysqlhotcopy db_name [/path/to/new_directory]
```

方法二：手工锁表COPY

```mysql
mysql >  flush tables for read;

-- 然后CP数据文件到备份目录下。

```



# 四、各种备份与恢复方法实现

## 1. `SELECT ... INTO OUTFILE`

利用`SELECT ... INTO OUTFILE`实现数据的备份与还原。

（1）备份数据

```mysql
mysql > use hellodb; --  打开 hellodb 库 
mysql > select * from students;  -- 查看 students 的属性
mysql > select * from students where Age > 30 into outfile ‘/tmp/stud.txt’;  -- 将年龄大于30的同学的信息备份出来
```



（2）查看备份数据

```shell
shell >  cd  /tmp 
shell >  cat stud.txt
```



（3）数据被破坏

>  会发现是个文本文件。所以不能直接导入数据库了。需要使用LOADDATAINFILE恢复回到MySQL服务器端，删除年龄大于30的用户，模拟数据被破坏，

```mysql
mysql > delete from students where Age > 30;
```



（4）恢复数据

```mysql
mysql > load data infile '/tmp/stud.txt' into table students;
```



## 2. `mysqldump`

利用mysqldump工具对数据进行备份和还原。

mysqldump常用来做温备，所以首先需要对想备份的数据施加读锁。



### 2.1 施加读锁的方式

（1）直接在备份的时候添加选项

> - lock-all-tables：是对要备份的数据库的所有表施加读锁。
> - lock-table：仅对单张表施加读锁，即使是备份整个数据库，它也是在备份某张表的时候才对该表施加读锁，因此适用于备份单张表。



（2）在服务器端书写命令

```mysql
mysql > flush tables with read lock;  -- 施加锁，表示把位于内存上的表统统都同步到磁盘上去，然后施加读锁
mysql > flush tables with read lock;  -- 释放读
```

但这对于InnoDB存储引擎来讲，虽然也能够请求到读锁，但是不代表它的所有数据都已经同步到磁盘上，

因此当面对InnoDB的时候，要使用如下命令，看看InnoDB所有的数据都已经同步到磁盘上，才进行备份操作。

```mysql
mysql >  show engine innodb status;
```



### 2.2 备份过程

采取的备份策略为：完全备份+增量备份+二进制日志



（1）先给数据库做完全备份：

```shell
shell >  mysqldump -uroot --single-transaction --master-data = 2 --databases hellodb > /backup/hellodb_`date+%F`.sql
```

> - `--single-transaction`：基于此选项能实现热备InnoDB表，因此，不需要同时使用`--lock-all-tables`；
> - `--master-data=2`：记录备份那一时刻的二进制日志的位置，并且注释掉，1是不注释的；
> - `--databases hellodb`：指定备份的数据库。



（2）回到MySQL服务器端更新数据

```mysql
mysql > create table tb1( id int);  -- 创建表
mysql > insert into tb1 values( 1),( 2),( 3);   -- 插入数据，这里只做演示，随便插入了几个数
```



（3）先查看完全备份文件里边记录的位置

```shell
shell > cat hellodb_2013-09-08.sql | less 

--CHANGE MASTER TO MASTER_LOG_FILE =' mysql-bin.000013', MASTER_LOG_POS = 15684;  -- 记录了二进制日志的位
```







显示此时的二进制日志的位置，从备份文件里边记录的位置到我们此时的位置，即为增量的部







# 四、演示 `SELECT ... INTO OUTFILE` 备份与恢复

## 1. 准备数据库以及数据表



- **studymysql.sql**

```mysql
create database if not exists studymysql;
use studymysql;
create table if not exists user (
	id bigint ( 12 ) not null auto_increment,
	user_name varchar ( 45 ) not null,
	password varchar ( 45 ) not null,
    sex varchar ( 4 ) not null,
	age int ( 4 ) unsigned default null,
	city varchar ( 45 ) default null,
	start_date date default null,
	end_date date default null,
	description varchar ( 200 ) default null,
	primary key ( id ) 
);

insert into user (user_name, password,sex,age,city) values ('tom' ,'123','男',11,'shanghai');
insert into user (user_name, password,sex,age,city) values ('jack' ,'123','男',21,'shanghai');
insert into user (user_name, password,sex,age,city) values ('mike' ,'123','男',31,'shanghai');
insert into user (user_name, password,sex,age,city) values ('caly' ,'123','女',31,'shanghai');
insert into user (user_name, password,sex,age,city) values ('carter' ,'123','男',41,'wuhan');
insert into user (user_name, password,sex,age,city) values ('bush' ,'123','男',51,'wuhan');
insert into user (user_name, password,sex,age,city) values ('jerry' ,'123','男',61,'wuhan');
insert into user (user_name, password,sex,age,city) values ('xiaohong' ,'123','女',41,'wuhan');
insert into user (user_name, password,sex,age,city) values ('xiaoming' ,'123','男',41,'beijing');
```



登录shell，创建 studymysql.sql

```bash
[ray@localhost mysql]$ pwd
/home/ray/dev/mysql
[ray@localhost mysql]$ vim studymysql.sql

# 将上述文件粘贴，并保存

```



登录mysql

```shell
shell > mysql -uroot -p
```



查看数据库列表

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| example            |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)
```



准备 `studymysql` 数据库： 使用 source 命令运行sql

```mysql
mysql> source /home/ray/dev/mysql/studymysql.sql
Query OK, 1 row affected (0.00 sec)

Database changed
Query OK, 0 rows affected (0.08 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.00 sec)

-- 查看数据库列表，发现已经有 studymysql 数据库了
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| example            |
| mysql              |
| performance_schema |
| studymysql         |
+--------------------+
5 rows in set (0.00 sec)

mysql> use studymysql  			-- 切换数据库
Database changed

mysql> show tables;    		 	-- 查看数据表列表
+----------------------+
| Tables_in_studymysql |
+----------------------+
| user                 |
+----------------------+
1 row in set (0.00 sec)

mysql> select * from user;  	 -- 查看user 表
+----+-----------+----------+-----+------+----------+------------+----------+-------------+
| id | user_name | password | sex | age  | city     | start_date | end_date | description |
+----+-----------+----------+-----+------+----------+------------+----------+-------------+
|  1 | tom       | 123      | 男  |   11 | shanghai | NULL       | NULL     | NULL        |
|  2 | jack      | 123      | 男  |   21 | shanghai | NULL       | NULL     | NULL        |
|  3 | mike      | 123      | 男  |   31 | shanghai | NULL       | NULL     | NULL        |
|  4 | caly      | 123      | 女  |   31 | shanghai | NULL       | NULL     | NULL        |
|  5 | carter    | 123      | 男  |   41 | wuhan    | NULL       | NULL     | NULL        |
|  6 | bush      | 123      | 男  |   51 | wuhan    | NULL       | NULL     | NULL        |
|  7 | jerry     | 123      | 男  |   61 | wuhan    | NULL       | NULL     | NULL        |
|  8 | xiaohong  | 123      | 女  |   41 | wuhan    | NULL       | NULL     | NULL        |
|  9 | xiaoming  | 123      | 男  |   41 | beijing  | NULL       | NULL     | NULL        |
+----+-----------+----------+-----+------+----------+------------+----------+-------------+
9 rows in set (0.00 sec)

```



## 2. 备份数据

```
select * from 
```









# 参考资料

1. [MySQL 备份和恢复机制](https://segmentfault.com/a/1190000011980376)
2. [Mysql 数据库备份与恢复](http://blog.51cto.com/liwenjia/1936955)
3. [MySQL数据库备份&还原-LINUX](https://www.jianshu.com/p/b77dfd6d998b)
4. [MySQL 导入数据-菜鸟教程](http://www.runoob.com/mysql/mysql-database-import.html)
5. [LOAD DATA INFILE 导入数据](https://www.jianshu.com/p/bcafd8f3ad8e)

