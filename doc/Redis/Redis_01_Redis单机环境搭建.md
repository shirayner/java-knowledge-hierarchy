[TOC]







# 前言



# 一、Redis 单机环境搭建

## 1.下载解压

前往[官网](https://redis.io/download)下载稳定版



然后解压

```bash
# 创建安装目录
sudo mkdir /usr/local/redis/

# 解压至安装目录
sudo tar -zxvf redis-5.0.5.tar.gz  -C /usr/local/redis/
```







## 2.添加软链接

创建一个redis目录的软链接，这样做是为了不把redis目录固定在指定版本上，有利于Redis未来版本升级，算是安装软件的一种好习惯

(有疑问)

```bash
sudo ln -s /usr/local/redis/redis-5.0.5  redis
```



## 3.编译和安装

```bash
cd /usr/local/redis/redis-5.0.5

# 编译
make

# 安装
make install
```



## 3.启动redis

```bash
cd /usr/local/redis/redis-5.0.5/src
./redis-server
./redis-server ../redis.conf
```







# 参考资料

1. [菜鸟教程_Redis 安装](https://www.runoob.com/redis/redis-install.html)
2. 



