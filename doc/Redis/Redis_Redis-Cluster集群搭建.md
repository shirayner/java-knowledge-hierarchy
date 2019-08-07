[TOC]







# 前言



# 一、Redis Sentinel部署

## 1.部署Redis数据结点

### 1.1 启动主节点

（1）配置：redis-6379.conf

```properties
port 6379
daemonize yes
logfile ./log/6379.log
dbfilename dump-6379.rdb
dir ./data
```



（2）启动主节点

```bash
./redis-server redis-6379.conf
```



（3）验证主节点是否启动

```bash
 ./redis-cli -h 127.0.0.1 -p 6379 ping
```



### 1.2 启动两个从节点

（1）配置

redis-6380.conf

```properties
port 6380
daemonize yes
logfile ./log/6380.log
dbfilename dump-6380.rdb
dir ./data
slaveof 127.0.0.1 6379
```



从节点 redis-6381.conf 同理



（2）启动从节点

使用如下命令，分别启动 6380、6381从节点

```bash
./redis-server redis-6380.conf
```



（3）验证

```bash
 ./redis-cli -h 127.0.0.1 -p 6380 ping
 ./redis-cli -h 127.0.0.1 -p 6381 ping
```





### 1.3 确认主从关系

主节点视角：

```bash
λ ./redis-cli -h 127.0.0.1 -p 6379 info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6380,state=online,offset=1781,lag=0
slave1:ip=127.0.0.1,port=6381,state=online,offset=1781,lag=0
master_repl_offset:1781
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:1780
```



从节点视角：

```bash
λ ./redis-cli -h 127.0.0.1 -p 6380 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:2
master_sync_in_progress:0
slave_repl_offset:1837
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```





## 2.部署Sentinel结点

（1）配置

3个Sentinel节点的部署方法是完全一致的（端口不同）

redis-sentinel-26379.conf

```
port 26379
daemonize yes
logfile ./log/26379.log
dir ./data
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```



> 配置主节点为 127.0.0.1 6379 ，主节点别名为 mymaster，2代表判断主节点失败至少需要2个Sentinel节点同意



（2）启动sentinel

使用如下命令，分别启动 26379、26380、26381 sentinel节点

```bash
./redis-server redis-sentinel-26379.conf --sentinel
```





（3）确认

```bash
λ ./redis-cli -h 127.0.0.1 -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=3
```

























# 参考资料

