[TOC]





# 前言

- 在微服务架构中，业务都会被拆分成一个独立的服务，服务之间的通讯是基于http restful的

- Ribbon可以很好地控制HTTP和TCP客户端的行为
- Spring Cloud有两种调用方式，一种是 Ribbon + RestTemplate，另一种是Feign（集成Ribbon+Hystrix），本章主要讲解Ribbon



# Ribbon 简介

Ribbon 是一个客户端负载均衡器，它可以很好地控制HTTP和TCP客户端的行为，它具有如下功能：

> - 丰富的负载均衡策略
> - 重试机制
> - 支持多协议的异步和响应式模型
> - 容错
> - 缓存和批处理功能



关于服务端负载均衡与客户端负载均衡：

> - 负载均衡（LB，Load balancing）：即利用特定方式将流量分摊到多个操作单元上的一种手段，它对系统吞吐量和系统处理能力有着质的提升
>
> 
>
> - 对负载均衡的分类有：
>     - 从软硬件上分：
>         - **硬负载**：采用硬件设备（硬件负载均衡器）来提供负载均衡，代表有 F5负载均衡
>         - **软负载**：通过在服务器上安装的特定的负载均衡软件或是自带负载均衡模块完成对请求的分配派发，代表有Nginx负载均衡
>     - 从服务端和客户端上分
>         - **服务端负载均衡**：也叫集中式负载均衡，特指位于因特网和服务提供者之间，并负责把网络请求转发到各个提供者单位，这是Nginx 和 F5 就可以划为一类了
>         - **客户端负载均衡**：也叫进程内负载均衡，特指从一个实例库选取一个实例进行流量导入，在微服务的范畴内，实例库一般存储在 Eureka 、Consul、Zookeeper、etcd这样的注册中心，而此时的负载均衡器类似Ribbon的IPC(Inter-Process Communication，进程间通信)组件。服务提供方启动时，会将服务地址注册到服务注册中心，并通过心跳来维护，当服务消费者需要访问某个服务时，通过内置的LB组件定期向服务注册中心获取服务注册表，并以某种负载均衡策略选定一个目标服务地址，并发起请求。
>
> 
>
> - 常见的负载均衡算法，请参见：[五月的仓颉__几种简单的负载均衡算法及其Java代码实现](https://www.cnblogs.com/xrq730/p/5154340.html)：
>     - **轮询（Round Robin）**: 将请求按照顺序轮流的分配到服务器上，如 123，123，123
>     - **随机（Random）**： 通过系统随机函数，根据后端服务器列表的大小值来随机选择其中一台进行访问
>     - **源地址哈希（Hash）**：源地址哈希的思想是获取客户端访问的IP地址值，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是要访问的服务器的序号。
>     - **加权轮询（Weight Round Robin）**: 不同的服务器可能机器配置和当前系统的负载并不相同，因此它们的抗压能力也不尽相同，给配置高、负载低的机器配置更高的权重，让其处理更多的请求，而低配置、高负载的机器，则给其分配较低的权重，降低其系统负载
>     - **加权随机（Weight Random）**: 与加权轮询法类似，加权随机法也是根据后端服务器不同的配置和负载情况来配置不同的权重。不同的是，它是按照权重来随机选择服务器的，而不是顺序。
>     - **最小连接数（Least Connections）**:



# 一、创建服务消费者

## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringCloud_01_Discovery_01_Eureka入门示例](./SpringCloud_01_Discovery_01_Eureka入门示例.md)

子模块信息如下：

```groovy
group = 'com.ray.study'
artifact ='spring-cloud-02-consumer-ribbon'
```



## 2.引入依赖

### 2.1 继承父工程依赖

在父工程`spring-cloud-seeds` 的 `settings.gradle`加入子工程

```groovy
rootProject.name = 'spring-cloud-seeds'
include 'spring-cloud-01-discovery-01-eureka-server'
include 'spring-cloud-01-discovery-01-eureka-client'
include 'spring-cloud-01-discovery-02-consul-client'
include 'spring-cloud-02-consumer-ribbon'
```



这样，子工程`spring-cloud-02-consumer-ribbon`就会自动继承父工程中`subprojects` 函数里声明的项目信息





### 2.2 引入 consumer-ribbon 依赖

将子模块`spring-cloud-02-consumer-ribbon` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // eureka client
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'

    // ribbon
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-ribbon'

}

```



这里我们创建的服务消费者，也是一个 eureka client，同时集成了ribbon 来进行负载均衡

## 3. 修改配置

### 3.1 修改`application.yml`

主要配置 eureka client 

```yml
server:
  port: 9000

spring:
  application:
    name: consumer-ribbon   #指定服务名

eureka:
  instance:
    prefer-ip-address: true
  client:
    registerWithEureka: true   #服务注册开关
    fetchRegistry: true        #服务发现开关
    serviceUrl:   #Eureka客户端与Eureka服务端进行交互的地址，多个中间用逗号分隔
      defaultZone: http://localhost:8761/eureka/    # 指定 Eureka Server 地址

```



### 3.2 启用服务发现

在启动类上添加`@EnableDiscoveryClient`注解即可启用服务发现

```java
package com.ray.study.springcloud02consumerribbon;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class SpringCloud02ConsumerRibbonApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloud02ConsumerRibbonApplication.class, args);
	}

}

```



### 3.3 配置ribbon

配置 ribbon 进行负载均衡控制，同时也可设置自己需要的负载均衡算法

```java
package com.ray.study.springcloud02consumerribbon.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * description
 *
 * @author shira 2019/05/22 16:39
 */
@Configuration
public class RibbonConfig {

	/**
	 *  通过添加 @LoadBalanced 注解 声明该 restTemplate 用于负载均衡
	 * @return
	 */
	@Bean
	@LoadBalanced
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}

	/**
	 * 设置Ribbon采取的负载均衡算法
	 * @return
	 */
//	@Bean
//	public IRule ribbonRule(){
//		// 随机算法
//		return new RandomRule();
//	}

}

```





## 4.业务实现

这里我们写一个简单的controller，来调用前面创建的  eureka-client 服务

```java
package com.ray.study.springcloud02consumerribbon.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * description
 *
 * @author shira 2019/05/22 17:01
 */
@RestController
@Slf4j
public class HelloRibbonController {

	@Autowired
	RestTemplate restTemplate;

	@GetMapping("/hello")
	public String sayHello(){
		String result = restTemplate.getForObject("http://eureka-client/hello",String.class);
		log.info(result);
		return result;
	}

}

```



# 二、负载均衡测试

下面，我们将使用  [SpringCloud_01_Discovery_01_Eureka入门示例.md](./SpringCloud_01_Discovery_01_Eureka入门示例.md)  这一节中创建的 Eureka 注册中心和Eureka Client 服务提供者来进行演示

## 1.启动服务

我们进行下服务的启动配置（主要是指定各服务启动的端口）：

（1）注册中心 EurekaServer-8761

![1558520594730](images/1558520594730.png)



（2）服务提供者 EurekaClient 

这里我们启动三个  EurekaClient，对应的端口分别为：8762、8763、8764

![1558520704231](images/1558520704231.png)



（4）服务消费者 ConsumerRibbon

![1558520771656](images/1558520771656.png)





配置完成后，依次启动服务注册中心、服务提供者、服务消费者，启动成功后，访问 Eureka 可视化界面：

> http://localhost:8761/

会发现 eureka-client 有三个副本，下面消费者将使用Ribbon去轮询这三个服务提供者

![1558520947264](images/1558520947264.png)



## 2.负载均衡测试

访问如下地址， consumer-ribbon 将请求 eureka-client 的  sayHello，给用户返回下图的一段话

> http://localhost:9000/hello



![1558521341908](images/1558521341908.png)



多次请求，会发现  eureka-client  服务被轮询了

![1558521415165](images/1558521415165.png)











# 参考资料

## 1.Ribbon

1. [一起来学SpringCloud之-服务消费者（Ribbon）](https://blog.battcn.com/2017/07/26/springcloud/dalston/spring-cloud-ribbon/)
2. [SpringCloud Ribbon+RestTemplate 服务负载均衡调用](https://www.phpsong.com/3713.html)
3.  [《重新定义Spring Cloud实战》](https://item.jd.com/12447280.html)

## 2.负载均衡

1. [瞬息之间__LB 负载均衡的层次结构](https://www.cnblogs.com/mindwind/p/5339657.html)
2. [常见的负载均衡算法](https://www.jianshu.com/p/1e56ace862e7)
3. [几种简单的负载均衡算法及其Java代码实现](https://www.cnblogs.com/xrq730/p/5154340.html)



