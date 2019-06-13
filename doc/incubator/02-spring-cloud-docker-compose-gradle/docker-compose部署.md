[TOC]







# 前言



# 使用Docker-compose部署项目

## 1.Docker-compose.yml



```yml
# docker-compose.yaml
version: "3"
services:
  register-server-0:
    container_name: register-server-0
    image: registry.cn-shanghai.aliyuncs.com/shirayner/dc-register-server-eureka:1.1.0
    hostname: register-server-0
    environment:
    - eureka.instance.hostname=localhost
    - server.port=8761
    ports:
    - "8761:8761"

  product-service-0:
    container_name: product-service-0
    image: registry.cn-shanghai.aliyuncs.com/shirayner/dc-product-service:1.1.0
    hostname: product-service-0
    ports:
    - "8771:8771"
    links:
    - redis
    - mysql

  order-service-0:
    container_name: order-service-0
    image: registry.cn-shanghai.aliyuncs.com/shirayner/dc-order-service:1.1.0
    hostname: order-service-0
    ports:
    - "8781:8781"
    links:
    - redis
    - mysql

  redis:
    container_name: redis
    hostname: redis
    image: redis:4.0.11
    ports:
    - "6379:6379"
    expose:
    - "6379"

  mysql:
    container_name: mysql
    hostname: mysql
    image: mysql:5.7.17
    ports:
    - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
    - ./mysql/mysql_data:/var/lib/mysql
    - ./mysql/mysql_db.cnf:/etc/mysql/conf.d/mysql_db.cnf

```





## 2.mysql_db.cnf

```cnf
[mysqld]
user=mysql
default-storage-engine=INNODB
character-set-server=utf8
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
```





## 3.创建数据库

启动mysql

```
docker-compose up 
```

进入 mysql 容器

> **On Windows CMD (not switching to bash)**
>
> ```
> docker exec -it <container-id> /bin/sh
> ```
>
> **On Windows CMD (after switching to bash)**
>
> ```
> docker exec -it <container-id> //bin//sh
> ```
>
> or
>
> ```
> winpty docker exec -it <container-id> //bin//sh
> ```
>
> **On Git Bash**
>
> ```
> winpty docker exec -it <container-id> //bin//sh
> ```



创建数据库

```
CREATE DATABASE IF NOT EXISTS dc_product_service DEFAULT CHARACTER SET utf8;
CREATE DATABASE IF NOT EXISTS dc_order_service DEFAULT CHARACTER SET utf8;
```

















# 参考资料

1. [使用docker-compose配置mysql数据库并且初始化用户](https://www.cnblogs.com/mmry/p/8812599.html)



