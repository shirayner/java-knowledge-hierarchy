# 查询优化实例_分页查询优化

[toc]



## 推荐阅读

> - [如果谁再问你“如何优化mysql分页查询”，请把这篇文章甩给他](https://www.modb.pro/db/25854)
> - [4种MySQL分页查询优化的方法，你知道几个？](https://juejin.cn/post/6844903955470745614)
> - [MySQL——优化嵌套查询和分页查询](https://www.cnblogs.com/songwenjie/p/9563763.html)
> - [MySQL的limit用法和分页查询的性能分析及优化](https://segmentfault.com/a/1190000008859706)



## 数据库准备

创建一个数据库，然后做如下操作：

### 1.创建数据表

```mysql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `username` varchar(255) NOT NULL DEFAULT '' COMMENT 'User name',
  `password` varchar(255) NOT NULL DEFAULT '' COMMENT 'password',
  `email` varchar(255) NOT NULL DEFAULT '' COMMENT 'email',
  `age` int(11) NOT NULL DEFAULT '-1' COMMENT 'age',
  `create_time` timestamp(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT 'Create time',
  `update_time` timestamp(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT 'update time',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```



### 2.添加数据

```mysql
-- 1.创建批量插入数据的存储过程
DROP PROCEDURE IF exists BatchInsertUser;
delimiter //
CREATE PROCEDURE BatchInsertUser(IN offset INT, IN num INT) 
begin
    declare i int; 
    set i = offset;
    while i <= num do 
        insert into user( `username`, `password`, `email`, `age` ) values( concat('tom', i), '123456', concat('tom', i, '@qq.com'), i );
        set i=i+1;
    end while;
end //
delimiter ;

-- 2.调用存储过程
CALL BatchInsertUser(1, 10000020);  
```

写一个 mysql 存储过程来插入1千万条数据 









## 一、分页查询过慢的原因

### 1.分页查询语法

参考官网：

> - [MySQL 8.0 Reference Manual/.../SELECT Statement](https://dev.mysql.com/doc/refman/8.0/en/select.html)
> - 

```mysql
SELECT * 
FROM table 
[LIMIT {[offset,] row_count | row_count OFFSET offset}]
```

`LIMIT`子句可用于约束[`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/select.html)语句返回的行数，有两个参数：

> - offset：指定要返回的第一行的偏移量，初始行的偏移量是0（不是1）
> - row_count：指定要返回的最大行数，row_count为-1表示检索从偏移量开始的第一行到最后一行的所有结果。
> - 默认按主键排序



注意事项：

> LIMIT查询过程为：先查询排序并取 offset + N行，然后丢弃掉前 offset 行，最后返回 N 行。



### 2.分页查询过慢的现象

偏移量越大，查询时间越久。

演示走聚集索引时的分页查询的耗时情况：

```mysql
-- 以下示例的查询耗时取最小值（这里不要求特别精确，按照一个标准取一个大概的值即可）

-- offset=0, cost=1ms
select * from user order by id limit 0,10;  

-- offset=1000, cost=2ms
select * from user order by id limit 1000,10; 

-- offset=10,000, cost=7ms
select * from user order by id limit 10000,10; 

-- offset=100,000, cost=48ms
select * from user order by id limit 100000,10; 

-- offset=1,000,000, cost=671ms
select * from user order by id limit 1000000,10; 

-- offset=10,000,000, cost=6.467s
select * from user order by id limit 10000000,10; 
```





### 3.分页查询过慢的原因

（1）偏移量为 0

```mysql
explain select * from user order by id limit 0,10;  
```

| id   | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra |      |
| ---- | ----------- | ----- | ---------- | ----- | ------------- | ------- | ------- | ---- | ---- | -------- | ----- | ---- |
| 1    | SIMPLE      | user  |            | index |               | PRIMARY | 8       |      | 10   | 100      |       |      |

（2）偏移量为 1,000

```mysql
explain select * from user order by id limit 1000,10;
```

| id   | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra |      |
| ---- | ----------- | ----- | ---------- | ----- | ------------- | ------- | ------- | ---- | ---- | -------- | ----- | ---- |
| 1    | SIMPLE      | user  |            | index |               | PRIMARY | 8       |      | 1010 | 100      |       |      |

（3）偏移量为 10,000

```mysql
explain select * from user order by id limit 10000,10;
```

| id   | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows  | filtered | Extra |      |
| ---- | ----------- | ----- | ---------- | ----- | ------------- | ------- | ------- | ---- | ----- | -------- | ----- | ---- |
| 1    | SIMPLE      | user  |            | index |               | PRIMARY | 8       |      | 10010 | 100      |       |      |

（4）偏移量为 100,000

```mysql
explain select * from user order by id limit 100000,10;
```

| id   | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows   | filtered | Extra |      |
| ---- | ----------- | ----- | ---------- | ----- | ------------- | ------- | ------- | ---- | ------ | -------- | ----- | ---- |
| 1    | SIMPLE      | user  |            | index |               | PRIMARY | 8       |      | 100010 | 100      |       |      |

（5）偏移量为 1,000,000

```mysql
explain select * from user order by id limit 1000000,10;
```

| id   | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows    | filtered | Extra |      |
| ---- | ----------- | ----- | ---------- | ----- | ------------- | ------- | ------- | ---- | ------- | -------- | ----- | ---- |
| 1    | SIMPLE      | user  |            | index |               | PRIMARY | 8       |      | 1000010 | 100      |       |      |

explain 执行计划包含的信息如下：

> - id：标识符,表示执行顺序.
> - select_type：查询类型。
> - table：查询的表。
> - partitions：使用的哪个分区,需要结合表分区才能看到(since 5.7)
> - type：查询类型,例如index索引
> - possible_key：可能用到的索引,如果是多个索引以逗号隔开
> - key：实际用到的索引,保存的是索引的名称,如果是多个索引以逗号隔开
> - key_len：使用到索引的长度
> - ref：引用索引对应的表中哪些行
> - rows：表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数
> - filtered：按表条件过滤的行百分比
> - Extra：额外的信息,using file sort,using where

对于查询语句（5），表示此次查询用到了主键索引，并且一共扫描了 1,000,010 行数据，但是经过过滤之后，最终只返回了10条数据。

> - Backward index scan：反向扫描索引，当需要进行降序排列时，MySQL会反向扫描索引，这样最终结果直接就是降序的了，这样也就无需额外的排序了。





结论：

- Mysql 在进行分页查询时，会先扫描 并排序  offset+N 行数据，然后丢弃掉前 offset 行，最终返回 N 行数据。
- 这样，当 offset 特别大的时候，就会扫描太多无用数据，查询和排序的代价非常高，效率就会非常差。



## 二、分页查询优化

优化思想：避免扫描过多的记录

### 1.覆盖索引法

**在索引上完成排序分页的操作，最后根据主键关联回表查询所需要的其他列内容。**

> 参考：
>
> - [MySQL优化：如何避免回表查询？什么是索引覆盖？ (转)](https://www.cnblogs.com/myseries/p/11265849.html)
> - [回表与覆盖索引，索引下推](https://www.jianshu.com/p/d0d3de6832b9)
> - [MySQL索引的原理，B+树、聚集索引和二级索引的结构分析](https://juejin.cn/post/6844903919525740552)
>



> - 覆盖索引：InnoDB聚集索引的叶子节点存储行记录，如果索引中包含了查询中的所有字段（**覆盖索引**），那么就无需去扫描叶子节点获取具体的数据行，这样节省了很多时间
> - 回表：如果查询的字段中包含了非索引列，那么就需要去扫描叶子节点获取具体的数据行，这就是回表



```mysql
explain select * from user a inner join (select id from user order by id limit 1000000,10) b on a.id = b.id;
```

| id   | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref  | rows    | filtered | Extra       |      |
| ---- | ----------- | ---------- | ---------- | ------ | ------------- | ------- | ------- | ---- | ------- | -------- | ----------- | ---- |
| 1    | PRIMARY     | <derived2> |            | ALL    |               |         |         |      | 1000010 | 100      |             |      |
| 1    | PRIMARY     | a          |            | eq_ref | PRIMARY       | PRIMARY | 8       | b.id | 1       | 100      |             |      |
| 2    | DERIVED     | user       |            | index  |               | PRIMARY | 8       |      | 1000010 | 100      | Using index |      |

表示此次先走主键索引，获取10条id数据，然后形成衍生表, 然后和 a表关联再回表查询其他列内容。



```mysql
-- 优化前：offset=1,000,000, cost=497ms
select * from user order by id limit 1000000,10;

-- 优化后：offset=1,000,000, cost=216ms
select * from user a inner join (select id from user order by id limit 1000000,10) b on a.id = b.id;
```





### 2.最大id查询法

思路：记住上一页的最后一条Id=k，那么下一次只需要查询id>k的 N 条数据即可。

局限：只使用于自增主键，对于UUID生成的主键不适用

```mysql
explain select * from user where id > 1000000 order by id desc limit 10;
```

| id   | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows   | filtered | Extra                            |      |
| ---- | ----------- | ----- | ---------- | ----- | ------------- | ------- | ------- | ---- | ------ | -------- | -------------------------------- | ---- |
| 1    | SIMPLE      | user  |            | range | PRIMARY       | PRIMARY | 8       |      | 680828 | 100      | Using where; Backward index scan |      |

表示此次查询使用主键索引进行了范围扫描，会扫描id>100w的记录，后面还有 70w，这里的 rows 标识可能扫描了 680828 行，但是由于 limit 10 的存在，因此扫描 10 行之后，就停止了。



```mysql
-- 优化前：offset=1,000,000, cost=497ms
select * from user order by id limit 1000000,10;

-- 优化后：offset=1,000,000, cost=1ms
select * from user where id > 1000000 order by id desc limit 10;
```















