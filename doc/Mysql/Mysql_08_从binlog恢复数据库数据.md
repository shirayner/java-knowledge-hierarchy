# 从binlog恢复数据库数据

[toc]



## 推荐阅读

> - [使用mysql的binlog恢复误操作(update|delete)的数据](https://blog.csdn.net/aeroleo/article/details/77929917)
> - [Mysql 通过binlog日志恢复数据](https://www.cnblogs.com/YCcc/p/10825870.html)
> - [MySQL 通过 binlog 恢复数据](https://learnku.com/articles/20702)
> - 









## 命令清单

```
C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqlbinlog.exe  C:\dev-env\Mysql\mysql-5.7.27-winx64\data\mysql-bin.000001

C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqldump.exe -uroot -proot ailanni >C:\Users\shira\Desktop\Mess\ailanni.sql


C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysql -uroot -proot ailanni<C:\Users\shira\Desktop\Mess\ailanni.sql


C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqlbinlog.exe  -help

C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqlbinlog.exe  C:\dev-env\Mysql\mysql-5.7.27-winx64\data\mysql-bin.000005 >d:\1.txt


C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqlbinlog.exe  --start-date="2020-06-08 19:30:00" --stop-date="2020-06-08 22:00:00" C:\dev-env\Mysql\mysql-5.7.27-winx64\data\mysql-bin.000006 |mysql -uroot -proot


flush logs;
show master logs;


C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqlbinlog.exe --base64-output=decode-rows -v  C:\dev-env\Mysql\mysql-5.7.27-winx64\data\mysql-bin.000003


```





## 1.数据准备

### 1.1 开启binlog

查看是否开始binlog

```bash
mysql>  show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
1 row in set, 1 warning (0.00 sec)
```



若没有开启，则需要修改 `my.ini` ，开启binlog 

```properties
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8
[mysqld]
# 1 表示忽略大小写，0表示解析大小写
lower_case_table_names=1
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=C:\dev-env\Mysql\mysql-5.7.27-winx64
# 设置mysql数据库的数据的存放目录
datadir=C:\dev-env\Mysql\mysql-5.7.27-winx64\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 开启binlog
log-bin=mysql-bin
binlog-format=Row
server-id=1001
```



### 1.2 准备数据库

（1）准备如下数据库：

```sql
create database test_binlog;
use test_binlog;

-- ----------------------------
-- Structure of user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
  `id` int(11) NOT NULL,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES (1, 'tom1');
INSERT INTO `user` VALUES (2, 'tom2');
INSERT INTO `user` VALUES (3, 'tom3');
INSERT INTO `user` VALUES (4, 'tom4');
```





（2）全量备份数据库

```
C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqldump -uroot -proot test_binlog > D:\test_binlog.sql
```



## 2.数据误操作

### 2.1 新增数据

上次备份数据库之后，又新增了3条数据

```sql
INSERT INTO `user` VALUES (5, 'tom5');
INSERT INTO `user` VALUES (6, 'tom6');
INSERT INTO `user` VALUES (7, 'tom7');
```



### 2.2 删库

删除数据库





## 3.数据库恢复

发现数据异常之后，首先停止mysql对外服务，然后尽快恢复数据

### 3.1 刷新binlog

```sql
flush logs;
```

### 3.2 查看binlog日志内容

（1）确定日志刷新到了哪个日志文件中

```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

查出来的是 mysql-bin.000002 ，但是上面的新增数据的日志却是记在 mysql-bin.000001 中



（2）查看 binlog 日志内容

```
C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqlbinlog.exe --base64-output=decode-rows -v  C:\dev-env\Mysql\mysql-5.7.27-winx64\data\mysql-bin.000001 >D:binlog.txt
```

内容如下：

```
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#200609  0:06:44 server id 1001  end_log_pos 123 CRC32 0x6455468b 	Start: binlog v 4, server v 5.7.27-log created 200609  0:06:44
# at 123
#200609  0:06:44 server id 1001  end_log_pos 154 CRC32 0xec2b6a24 	Previous-GTIDs
# [empty]
# at 154
#200609  0:07:04 server id 1001  end_log_pos 219 CRC32 0xfcc4f5f3 	Anonymous_GTID	last_committed=0	sequence_number=1	rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 219
#200609  0:07:04 server id 1001  end_log_pos 360 CRC32 0x8e9bd5f5 	Query	thread_id=25	exec_time=0	error_code=0
use `test_binlog`/*!*/;
SET TIMESTAMP=1591632424/*!*/;
SET @@session.pseudo_thread_id=25/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8mb4 *//*!*/;
SET @@session.character_set_client=45,@@session.collation_connection=45,@@session.collation_server=33/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
DROP TABLE IF EXISTS `user` /* generated by server */
/*!*/;
# at 360
#200609  0:07:04 server id 1001  end_log_pos 425 CRC32 0x14bef1d3 	Anonymous_GTID	last_committed=1	sequence_number=2	rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 425
#200609  0:07:04 server id 1001  end_log_pos 774 CRC32 0xcb88a198 	Query	thread_id=25	exec_time=0	error_code=0
SET TIMESTAMP=1591632424/*!*/;
CREATE TABLE `user`  (

  `id` int(11) NOT NULL,

  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,

  PRIMARY KEY (`id`) USING BTREE

) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic
/*!*/;
# at 774
#200609  0:07:04 server id 1001  end_log_pos 839 CRC32 0x57325aad 	Anonymous_GTID	last_committed=2	sequence_number=3	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 839
#200609  0:07:04 server id 1001  end_log_pos 918 CRC32 0x7588143b 	Query	thread_id=25	exec_time=0	error_code=0
SET TIMESTAMP=1591632424/*!*/;
BEGIN
/*!*/;
# at 918
#200609  0:07:04 server id 1001  end_log_pos 975 CRC32 0x5895b86f 	Table_map: `test_binlog`.`user` mapped to number 125
# at 975
#200609  0:07:04 server id 1001  end_log_pos 1021 CRC32 0x6c81100d 	Write_rows: table id 125 flags: STMT_END_F
### INSERT INTO `test_binlog`.`user`
### SET
###   @1=1
###   @2='tom1'
# at 1021
#200609  0:07:04 server id 1001  end_log_pos 1052 CRC32 0xeb25a146 	Xid = 1358
COMMIT/*!*/;
# at 1052
#200609  0:07:04 server id 1001  end_log_pos 1117 CRC32 0x16e08c04 	Anonymous_GTID	last_committed=3	sequence_number=4	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1117
#200609  0:07:04 server id 1001  end_log_pos 1196 CRC32 0xcd874a31 	Query	thread_id=25	exec_time=0	error_code=0
SET TIMESTAMP=1591632424/*!*/;
BEGIN
/*!*/;
# at 1196
#200609  0:07:04 server id 1001  end_log_pos 1253 CRC32 0xa76f2efc 	Table_map: `test_binlog`.`user` mapped to number 125
# at 1253
#200609  0:07:04 server id 1001  end_log_pos 1299 CRC32 0x698dbef3 	Write_rows: table id 125 flags: STMT_END_F
### INSERT INTO `test_binlog`.`user`
### SET
###   @1=2
###   @2='tom2'
# at 1299
#200609  0:07:04 server id 1001  end_log_pos 1330 CRC32 0xfce65505 	Xid = 1359
COMMIT/*!*/;
# at 1330
#200609  0:07:04 server id 1001  end_log_pos 1395 CRC32 0x2e69f767 	Anonymous_GTID	last_committed=4	sequence_number=5	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1395
#200609  0:07:04 server id 1001  end_log_pos 1474 CRC32 0x29dc80cb 	Query	thread_id=25	exec_time=0	error_code=0
SET TIMESTAMP=1591632424/*!*/;
BEGIN
/*!*/;
# at 1474
#200609  0:07:04 server id 1001  end_log_pos 1531 CRC32 0x320bc644 	Table_map: `test_binlog`.`user` mapped to number 125
# at 1531
#200609  0:07:04 server id 1001  end_log_pos 1577 CRC32 0xcb6d04d8 	Write_rows: table id 125 flags: STMT_END_F
### INSERT INTO `test_binlog`.`user`
### SET
###   @1=3
###   @2='tom3'
# at 1577
#200609  0:07:04 server id 1001  end_log_pos 1608 CRC32 0xdcef7392 	Xid = 1360
COMMIT/*!*/;
# at 1608
#200609  0:07:04 server id 1001  end_log_pos 1673 CRC32 0xfb1b83be 	Anonymous_GTID	last_committed=5	sequence_number=6	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1673
#200609  0:07:04 server id 1001  end_log_pos 1752 CRC32 0x1224a7a3 	Query	thread_id=25	exec_time=0	error_code=0
SET TIMESTAMP=1591632424/*!*/;
BEGIN
/*!*/;
# at 1752
#200609  0:07:04 server id 1001  end_log_pos 1809 CRC32 0xcdaf7615 	Table_map: `test_binlog`.`user` mapped to number 125
# at 1809
#200609  0:07:04 server id 1001  end_log_pos 1855 CRC32 0xa1c92a83 	Write_rows: table id 125 flags: STMT_END_F
### INSERT INTO `test_binlog`.`user`
### SET
###   @1=4
###   @2='tom4'
# at 1855
#200609  0:07:04 server id 1001  end_log_pos 1886 CRC32 0x53a3dd53 	Xid = 1361
COMMIT/*!*/;
# at 1886
#200609  0:07:23 server id 1001  end_log_pos 1951 CRC32 0xa01f8a88 	Anonymous_GTID	last_committed=6	sequence_number=7	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1951
#200609  0:07:23 server id 1001  end_log_pos 2030 CRC32 0x6206c951 	Query	thread_id=25	exec_time=0	error_code=0
SET TIMESTAMP=1591632443/*!*/;
BEGIN
/*!*/;
# at 2030
#200609  0:07:23 server id 1001  end_log_pos 2087 CRC32 0x76bdbb04 	Table_map: `test_binlog`.`user` mapped to number 125
# at 2087
#200609  0:07:23 server id 1001  end_log_pos 2133 CRC32 0xd325ddfb 	Write_rows: table id 125 flags: STMT_END_F
### INSERT INTO `test_binlog`.`user`
### SET
###   @1=5
###   @2='tom5'
# at 2133
#200609  0:07:23 server id 1001  end_log_pos 2164 CRC32 0xb3bc4e7b 	Xid = 1392
COMMIT/*!*/;
# at 2164
#200609  0:07:23 server id 1001  end_log_pos 2229 CRC32 0x790753f8 	Anonymous_GTID	last_committed=7	sequence_number=8	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2229
#200609  0:07:23 server id 1001  end_log_pos 2308 CRC32 0x12642d91 	Query	thread_id=25	exec_time=0	error_code=0
SET TIMESTAMP=1591632443/*!*/;
BEGIN
/*!*/;
# at 2308
#200609  0:07:23 server id 1001  end_log_pos 2365 CRC32 0x33581063 	Table_map: `test_binlog`.`user` mapped to number 125
# at 2365
#200609  0:07:23 server id 1001  end_log_pos 2411 CRC32 0xcf3b4e2a 	Write_rows: table id 125 flags: STMT_END_F
### INSERT INTO `test_binlog`.`user`
### SET
###   @1=6
###   @2='tom6'
# at 2411
#200609  0:07:23 server id 1001  end_log_pos 2442 CRC32 0x00079e47 	Xid = 1393
COMMIT/*!*/;
# at 2442
#200609  0:07:23 server id 1001  end_log_pos 2507 CRC32 0x33e375df 	Anonymous_GTID	last_committed=8	sequence_number=9	rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2507
#200609  0:07:23 server id 1001  end_log_pos 2586 CRC32 0x9c6081fa 	Query	thread_id=25	exec_time=0	error_code=0
SET TIMESTAMP=1591632443/*!*/;
BEGIN
/*!*/;
# at 2586
#200609  0:07:23 server id 1001  end_log_pos 2643 CRC32 0x9808eeb0 	Table_map: `test_binlog`.`user` mapped to number 125
# at 2643
#200609  0:07:23 server id 1001  end_log_pos 2689 CRC32 0x70c03a7d 	Write_rows: table id 125 flags: STMT_END_F
### INSERT INTO `test_binlog`.`user`
### SET
###   @1=7
###   @2='tom7'
# at 2689
#200609  0:07:23 server id 1001  end_log_pos 2720 CRC32 0x2c115729 	Xid = 1394
COMMIT/*!*/;
# at 2720
#200609  0:07:37 server id 1001  end_log_pos 2785 CRC32 0xc5d8a341 	Anonymous_GTID	last_committed=9	sequence_number=10	rbr_only=no
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 2785
#200609  0:07:37 server id 1001  end_log_pos 2916 CRC32 0x6952c4bd 	Query	thread_id=32	exec_time=0	error_code=0
SET TIMESTAMP=1591632457/*!*/;
SET @@session.pseudo_thread_id=32/*!*/;
DROP TABLE `user` /* generated by server */
/*!*/;
# at 2916
#200609  0:07:51 server id 1001  end_log_pos 2963 CRC32 0x77e3f9cb 	Rotate to mysql-bin.000002  pos: 4
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

```



### 3.2 导入全量备份数据

先创建数据库

然后导入全量备份

```
mysql -uroot -proot test_binlog < D:\test_binlog.sql
```

此时查询数据，发现全量备份数据已经导入

```sql
mysql> select * from user;
+----+------+
| id | name |
+----+------+
|  1 | tom1 |
|  2 | tom2 |
|  3 | tom3 |
|  4 | tom4 |
+----+------+
4 rows in set (0.00 sec)
```



### 3.3 从binlog恢复增量数据

```
C:\dev-env\Mysql\mysql-5.7.27-winx64\bin\mysqlbinlog.exe  --start-datetime="20-06-09  0:07:23" --stop-datetime="20-06-09  0:07:24"  --database=test_binlog  C:\dev-env\Mysql\mysql-5.7.27-winx64\data\mysql-bin.000001 |mysql -uroot -proot
```



此时查询数据，发现增量数据已经恢复

```sql
mysql> select * from user;
+----+------+
| id | name |
+----+------+
|  1 | tom1 |
|  2 | tom2 |
|  3 | tom3 |
|  4 | tom4 |
|  5 | tom5 |
|  6 | tom6 |
|  7 | tom7 |
+----+------+
7 rows in set (0.00 sec)
```







