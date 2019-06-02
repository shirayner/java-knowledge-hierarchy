[TOC]





# 前言

> 在Spring Cloud 微服务中，各个服务提供者都是通过Http接口的形式对外提供服务，因此服务消费者调用服务提供者时，底层是通过 Http Client 的方式访问。
>
> 上一节，我们的服务间调用是使用了Spring 封装的 RestTemplate，其实RestTemplate 的底层也是使用的 Http Client 去访问的，常用的 Http Client 技术有： 
>
> - JDK 原生的 URLConnection
> - Apache 的 HttpClient
> - Square 的 OkHttp
> - Netty 的异步 HttpClient 等
>
> 这一节使用的 Feign ，在底层就是使用如上的HttpClient技术进行服务间Http调用的。



Spring Cloud Feign  是一种声明式、模板化的 HTTP 客户端，它使得编写 Web Service 客户端变得更加简单，我们只需要**通过创建接口并用注解来配置它既可完成对 Web 服务接口的绑定**。它具有如下特性：

> - 可插拔的注解支持，包括 Feign 注解、JAX-RS 注解
> - 支持可插拔的HTTP编码器和解码器
> - 支持 Hystrix 和它的 Fallback
> - 支持Ribbon的负载均衡
> - 支持Http请求和响应的压缩





# 一、创建服务消费者

## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringCloud_01_Discovery_01_Eureka入门示例](./SpringCloud_01_Discovery_01_Eureka入门示例.md)

子模块信息如下：

```groovy
group = 'com.ray.study'
artifact ='spring-cloud-03-consumer-feign'
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
include 'spring-cloud-03-consumer-feign'
```



这样，子工程`spring-cloud-03-consumer-feign`就会自动继承父工程中`subprojects` 函数里声明的项目信息





### 2.2 引入 consumer-feign 依赖

将子模块`spring-cloud-03-consumer-feign` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // eureka client
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'

    // feign
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'

}

```



这里我们创建的服务消费者，也是一个 eureka client

## 3. 修改配置

### 3.1 修改`application.yml`

主要配置 eureka client 

```yml
server:
  port: 8765

spring:
  application:
    name: consumer-feign    #指定服务名

eureka:
  instance:
    prefer-ip-address: true
  client:
    registerWithEureka: true   #服务注册开关
    fetchRegistry: true        #服务发现开关
    serviceUrl:   #Eureka客户端与Eureka服务端进行交互的地址，多个中间用逗号分隔
      defaultZone: http://localhost:8761/eureka/    # 指定 Eureka Server 地址




```



### 3.2 启用Feign

在启动类上

- 添加`@EnableDiscoveryClient`注解可启用服务发现
- 添加`@EnableFeignClients`注解可启用 Feign

```java
package com.ray.study.springcloud03consumerfeign;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;


@EnableFeignClients
@EnableEurekaClient
@SpringBootApplication
public class SpringCloud03ConsumerFeignApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloud03ConsumerFeignApplication.class, args);
	}

}

```







## 4.业务实现

### 4.1 dto

数据传输实体类

```java
package com.ray.study.springcloud03consumerfeign.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;

/**
 * description
 *
 * @author shira 2019/05/27 10:36
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {

	private Long id;

	private String name;

	private Integer age;

	private Date creationDate;

	private Date lastUpdateDate;

}

```





### 4.2 FeignClient

编写一个FeignClient用于调用远程服务提供者提供的服务

```java
package com.ray.study.springcloud03consumerfeign.service;

import com.ray.study.springcloud03consumerfeign.dto.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.*;

/**
 * description
 *
 * @author shira 2019/05/27 11:10
 */
@FeignClient(name = "eureka-client",path = "/user")
public interface UserService {

	@GetMapping("/{id}")
	String getUser1(@PathVariable("id") Long id);

	@GetMapping("/get2")
	String getUser2(@RequestParam("name") String name,  @RequestHeader("token") String token);

	@PostMapping("/get3")
	User getUser3(@RequestBody  User user);
	
}

```



Feign 多参数传递时，必须制定 @RequestParam、@RequestHeader注解的 value 值



### 4.3 UserController

```java
package com.ray.study.springcloud03consumerfeign.controller;

import com.ray.study.springcloud03consumerfeign.dto.User;
import com.ray.study.springcloud03consumerfeign.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.Date;

/**
 * description
 *
 * @author shira 2019/05/27 10:38
 */
@RestController
@RequestMapping("/user")
public class UserController {


	@Autowired
	UserService userService;

	/**
	 * get 路径请求参数传递
	 */
	@GetMapping("/get")
	public String getUser1(){
		return userService.getUser1(10001L);
	}

	/**
	 * get 请求参数传递
	 * @return
	 */
	@GetMapping("/get2")
	public String getUser2() {
		return userService.getUser2("张三", "271267312jhdsjfhdsjf3uwyruwe");
	}

	/**
	 *  post 请求参数传递
	 * @return
	 */
	@PostMapping("/get3")
	public User getUser3() {

		User user1 =  new User();
		user1.setId(10001L);
		user1.setName("张三");
		user1.setAge(22);
		user1.setCreationDate(new Date());

		return userService.getUser3(user1);

	}

}

```





## 5.测试

> 下面，我们将使用  [SpringCloud_01_Discovery_01_Eureka入门示例.md](./SpringCloud_01_Discovery_01_Eureka入门示例.md)  这一节中创建的 Eureka 注册中心和Eureka Client 服务提供者来进行演示



依次启动  

> - eurka-server： 服务注册中心
> - eureka-client：服务提供者
> - consumer-feign：本节创建的服务消费者



访问 consumer-feign 服务的 UserController 进行验证





# 参考资料

3.  [唐亚峰__一起来学SpringCloud之-服务消费者（Feign-上）](https://blog.battcn.com/2017/07/28/springcloud/dalston/spring-cloud-feign-up/#battcn-feign-hello)
2.  [方志鹏__SpringCloud教程第3篇：feign（F版本）](https://www.fangzhipeng.com/springcloud/2018/08/03/sc-f3-feign.html)
3. [《重新定义Spring Cloud实战》(F版)](https://item.jd.com/12447280.html)



