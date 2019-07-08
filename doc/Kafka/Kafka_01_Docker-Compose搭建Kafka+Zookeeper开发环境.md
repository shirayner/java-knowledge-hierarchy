[TOC]



# 前言



# 一、使用Docker-Compose部署Kafka集群

## 1.部署单节点伪分布式集群

> 单节点伪分布式集群是指集群由一台Zookeeper服务器和一台Kafka broker服务器组成

（1）编写docker-compose.yml

```yml
version: "3"
services:   
  zookeeper:
    image: zookeeper:3.4
    container_name: zookeeper
    hostname: zookeeper
    restart: always
    ports:
      - 2181:2181
    volumes:
      - ./zookeeper/data:/data
    networks:
      - baseNetwork

  kafka:
    image: wurstmeister/kafka:2.12-2.1.0
    container_name: kafka
    hostname: kafka
    restart: always
    ports:
      - 9092:9092
    networks:
      - baseNetwork
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - ./kafka/logs:/kafka

  kafka-manager:
    image: sheepkiller/kafka-manager
    container_name: kafka-manager
    hostname: kafka-manager
    ports:
      - 9000:9000
    networks:
      - baseNetwork
    environment:
      ZK_HOSTS: zookeeper:2181  


networks:
  baseNetwork:
    driver: bridge

```



执行如下命令

```bash
docker-compose up -d
```



（2）一个Zookeeper多个kafka的扩容

```
version: "3"
services:   
  zookeeper:
    image: zookeeper:3.4
    container_name: zookeeper
    hostname: zookeeper
    restart: always
    ports:
      - 2181:2181
    volumes:
      - ./zookeeper/data:/data
    networks:
      - baseNetwork

  kafka1:
    image: wurstmeister/kafka:2.12-2.1.0
    container_name: kafka1
    hostname: kafka1
    restart: always
    ports:
      - 9092:9092
    networks:
      - baseNetwork
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - ./kafka/logs:/kafka


  kafka2:
    image: wurstmeister/kafka:2.12-2.1.0
    container_name: kafka2
    hostname: kafka2
    restart: always
    ports:
      - 9093:9093
    networks:
      - baseNetwork
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - ./kafka/logs:/kafka

  kafka3:
    image: wurstmeister/kafka:2.12-2.1.0
    container_name: kafka3
    hostname: kafka3
    restart: always
    ports:
      - 9094:9094
    networks:
      - baseNetwork
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - ./kafka4/logs:/kafka


  kafka-manager:
    image: sheepkiller/kafka-manager
    container_name: kafka-manager
    hostname: kafka-manager
    ports:
      - 9000:9000
    networks:
      - baseNetwork
    environment:
      ZK_HOSTS: zookeeper:2181  


networks:
  baseNetwork:
    driver: bridge

```





```bash
docker-compose up -d 
```





## 3.部署多节点分布式集群

> 多节点Kafka集群由一套多节点Zookeeper集群和一套多节点Kafka集群组成
>
> 以下还有点问题

（1）编写 docker-compose.yml

```yml
version: "3"
services:   
  zk1:
    image: zookeeper:3.4
    container_name: zk1
    hostname: zk1
    restart: always
    ports:
      - 2181:2181
    volumes:
      - ./zk1/data:/data
    networks:
      - baseNetwork

  zk2:
    image: zookeeper:3.4
    restart: always
    ports:
      - 2182:2182
    volumes:
      - ./zk2/data:/data
    networks:
      - baseNetwork

  kafka:
    image: wurstmeister/kafka:2.12-2.1.0
    container_name: kafka
    hostname: kafka
    restart: always
    networks:
      - baseNetwork
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_ZOOKEEPER_CONNECT: zk1:2181,zk2:2182
    volumes:
      - ./kafka/logs:/kafka

  kafka-manager:
    image: sheepkiller/kafka-manager
    ports:
      - 9000:9000
    networks:
      - baseNetwork
    environment:
      ZK_HOSTS: zk1:2181,zk2:2182


networks:
  baseNetwork:
    driver: bridge

```







（2）执行命令：

````yml
docker-compose up -d --scale kafka=3
````







# 二、测试Kafka

进入 kafka docker 容器

```bash
docker exec -it kafka /bin/bash
```

进入kafka默认目录

```bash
cd /opt/kafka_2.12-2.1.0
```

会发现此目录中包含了许多kafka的可执行命令，下面我们将使用这些命令进行kafka的相关测试



## 1.创建topic

创建一个名为test的topic，该topic只有一个分区（partition），且该partition也只有一个副本（replication）处理消息

```bash
bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --topic test --partitions 1 --replication-factor 1
```



## 2.查看topic状态

```bash
bin/kafka-topics.sh --describe --zookeeper zookeeper:2181 --topic test
```



## 3.发送消息

```bash
bin/kafka-console-producer.sh --broker-list kafka:9092 --topic test
```



## 4.消费消息

```bash
bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic test --from-beginning
```



















# 参考资料

1. [kafka原理及Docker环境部署](http://kekefund.com/2018/10/26/kafka-docker/)
2. [docker-compose安装kafka集群和kafka-manager管理界面](https://blog.csdn.net/sinat_31908303/article/details/80447383)
3. [使用Docker快速搭建Zookeeper和kafka集群](https://www.cnblogs.com/once/p/10146666.html)
4. [docker-compose部署zk集群、kafka集群以及kafka-manager，及其遇到的问题和解决](https://www.cnblogs.com/jay763190097/p/10292227.html)
5. [docker-compose部署kafka和zookeeper(推荐)](https://blog.csdn.net/hechaojie_com/article/details/89521307)