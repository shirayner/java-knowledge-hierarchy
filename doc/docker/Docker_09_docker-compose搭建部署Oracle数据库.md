[TOC]



# 前言









# 一、环境准备

通常我们本地开发时需要准备某些服务，例如

> - redis
> - mysql

可采取 docker-compose 部署



## 1.docker-compose.yml

```yml
# docker-compose.yaml
# 各容器间通过服务名为默认的hostname进行访问，因此application.yml中要使用服务名
version: "3"
services:
  redis:
    image: redis 
    ports:
      - 6379:6379 
    restart: always
    command: redis-server --requirepass ${pwd} --notify-keyspace-events Eglx
    volumes:
      - ${REDIS_DIR}/conf:/usr/local/etc/redis
      - ${REDIS_DIR}/data:/data
    networks:
      - baseNetwork

  mysql:
    image: mysql:8.0.16
    ports:
      - "3306:3306"
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
    volumes:
      - ${MYSQL_DIR}/mysql_data:/var/lib/mysql
      - ${MYSQL_DIR}/mysql_db.cnf:/etc/mysql/conf.d/mysql_db.cnf
      - ${MYSQL_DIR}/init:/docker-entrypoint-initdb.d/
    networks:
      - baseNetwork
      
networks:
  baseNetwork:
    driver: bridge

```





## 2.配置文件

### 2.1 .env

```properties
# .env文件内容
# redis
REDIS_DIR=./redis

# mysql
MYSQL_DIR=./mysql
MYSQL_ROOT_PASSWORD=root
```





### 2.2 redis





### 2.3 mysql

（1）mysql配置文件

如docker-compose.yml所示，创建mysql配置文件`${MYSQL_DIR}/mysql_db.cnf`，如下所示

```properties
# mysql_db.cnf
[mysqld]
lower_case_table_names=1
character_set_server=utf8
max_connections=500
```



（2）mysql数据库初始化数据

若要在创建docker镜像时，对mysql初始化一些数据，则可以通过挂载如下卷来实现

```
 volumes:
      - ${MYSQL_DIR}/init:/docker-entrypoint-initdb.d/
```



docker会在创建镜像时，执行`/docker-entrypoint-initdb.d/`目录下的` .sql`文件。













# 参考资料

1. [使用docker-compose配置mysql数据库并且初始化用户](https://www.cnblogs.com/mmry/p/8812599.html)
2. 