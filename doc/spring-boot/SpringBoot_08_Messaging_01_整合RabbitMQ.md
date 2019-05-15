[TOC]



# 一、基本概念

## 1.Message Queue

MQ全称（Message Queue）又名消息队列，是一种异步通讯的中间件。

> - 可以将它理解成邮局，发送者将消息传递到邮局，然后由邮局帮我们发送给具体的消息接收者（消费者），具体发送过程与时间我们无需关心，它也不会干扰我进行其它事情。
> - **常见的MQ有kafka、activemq、zeromq、rabbitmq 等等，各大MQ的对比和优劣势可以自行Google**



## 2.RabbitMQ

**RabbitMQ是一个遵循AMQP协议**，由面向高并发的`erlanng`语言开发而成，用在实时的对可靠性要求比较高的消息传递上，支持多种语言客户端。支持`延迟队列（这是一个非常有用的功能）`….

### 2.1 基础概念

> - **Broker：**简单来说就是消息队列服务器实体
> - **Exchange：**消息交换机，它指定消息按什么规则，路由到哪个队列
> - **Queue：**消息队列载体，每个消息都会被投入到一个或多个队列
> - **Binding：**绑定，它的作用就是把`exchange`和`queue`按照路由规则绑定起来
> - **Routing Key：**路由关键字，`exchange`根据这个关键字进行消息投递
> - **vhost：**虚拟主机，一个`broker`里可以开设多个`vhost`，用作不同用户的权限分离
> - **producer：**消息生产者，就是投递消息的程序
> - **consumer：**消息消费者，就是接受消息的程序
> - **channel：**消息通道，在客户端的每个连接里，可建立多个`channel`，每个`channel`代表一个会话任务



### 2.2 RabbitMQ 官方文档中文译文

关于 RabbitMQ 的使用，可以阅读[梁桂钊](http://blog.720ui.com/)大大对官方文档的翻译

> - [【译】RabbitMQ 实战教程（一） Hello World!](http://blog.720ui.com/2017/rabbitmq_action_01_helloworld/)
> - [【译】RabbitMQ 实战教程（二） 工作队列](http://blog.720ui.com/2017/rabbitmq_action_02_workqueues/)
> - [【译】RabbitMQ 实战教程（三） 发布/订阅](http://blog.720ui.com/2017/rabbitmq_action_03_publish_subscribe/)
> - [【译】RabbitMQ 实战教程（四） 路由](http://blog.720ui.com/2017/rabbitmq_action_04_routing/)
> - [【译】RabbitMQ 实战教程（五） 主题](http://blog.720ui.com/2017/rabbitmq_action_05_topics/)



### 2.3 常见应用场景

> - **邮箱发送**：用户注册后投递消息到`rabbitmq`中，由消息的消费方异步的发送邮件，提升系统响应速度
> - **流量削峰**：一般在秒杀活动中应用广泛，秒杀会因为流量过大，导致应用挂掉，为了解决这个问题，一般在应用前端加入消息队列。用于控制活动人数，将超过此一定阀值的订单直接丢弃。缓解短时间的高流量压垮应用。
> - **订单超时**：利用`rabbitmq`的延迟队列，可以很简单的实现**订单超时**的功能，比如用户在下单后30分钟未支付取消订单
> - 还有更多应用场景就不一一列举了…..



# 二、准备RabbitMQ

**准备条件**：在SpringBoot整合RabbitMq之前需要确保已经正确安装和运行RabbitMQ，可参见 [RabbitMQ_01_安裝及配置](https://blog.csdn.net/qq_26981333/article/details/90238384)







# 三、SpringBoot 整合 RabbitMQ



Spring Boot 整合 RabbitMQ 是非常容易，只需要两个步骤：

> - 引入依赖
> - 配置 RabbitMQ 

## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringBoot_01_入门示例](./SpringBoot_01_入门示例.md)

```properties
group = 'com.ray.study'
artifact ='spring-boot-08-messaging-rabbitmq'
```



## 2.引入依赖

### 2.1 继承父工程依赖

在父工程`spring-boot-seeds` 的 `settings.gradle`加入子工程

```properties
rootProject.name = 'spring-boot-seeds'
include 'spring-boot-01-helloworld'
include 'spring-boot-02-restful-test'
include 'spring-boot-03-thymeleaf'
include 'spring-boot-04-swagger2'
include 'spring-boot-05-jpa'
include 'spring-boot-05-mybatis'
include 'spring-boot-05-tk-mybatis'
include 'spring-boot-06-nosql-redis'
include 'spring-boot-06-nosql-mongodb'
include 'spring-boot-07-cache-concurrentmap'
include 'spring-boot-07-cache-ehcache'
include 'spring-boot-07-cache-caffeine'
include 'spring-boot-07-cache-redis'
include 'spring-boot-08-messaging-rabbitmq'
```



这样，子工程`spring-boot-08-messaging-rabbitmq`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
		implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```



### 2.2 引入`RabbitMQ`依赖

将子模块`spring-boot-08-messaging-rabbitmq` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-amqp'
}

```





## 4. 配置RabbitMQ

### 4.1 修改`application.yml`



```yml
# 配置 rabbitmq:
spring:
  rabbitmq:
    publisher-confirms: true     # 是否确认发送的消息已经被消费

# 自定义配置：配置两个队列
rabbitmq:
  queue:
    msg: spingboot-queue-msg
    user: spingboot-queue-user

```



关于默认配置和可选配置可参考`spring-boot-autoconfigure` jar包中`spring-configuration-metadata.json`，这里选取部分默认配置：

```properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.publisher-confirms= false   
```



### 4.2 配置RabbitMQConfig

这里我们只是简单的注册两条队列

```java
package com.ray.study.springboot08messagingrabbitmq.config;

import org.springframework.amqp.core.Queue;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * RabbitMQ配置
 *
 * @author shira 2019/05/15 13:32
 */

@Configuration
public class RabbitMQConfig {

	@Value("${rabbitmq.queue.msg}")
	private String msgQueueName;

	@Value("${rabbitmq.queue.user}")
	private String userQueueName;
	
	@Bean
	public Queue defaultBookQueue() {
		// 第一个是 QUEUE 的名字,第二个是消息是否需要持久化处理
		return new Queue(msgQueueName, true);
	}

	@Bean
	public Queue manualBookQueue() {
		// 第一个是 QUEUE 的名字,第二个是消息是否需要持久化处理
		return new Queue(userQueueName, true);
	}
}
```





## 5.消息生产者

### 5.1 entity

```java
package com.ray.study.springboot08messagingrabbitmq.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.util.Date;

/**
 * description
 *
 * @author shira 2019/05/15 13:43
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User implements Serializable {

	private static final long serialVersionUID = -2164058270260403154L;

	private Long id;

	private String name;

	private Integer age;

	private Date creationDate;

	private Date lastUpdateDate;

}
```





### 5.2 service

（1）RabbitMsgService

```java
package com.ray.study.springboot08messagingrabbitmq.service;

import com.ray.study.springboot08messagingrabbitmq.entity.User;

/**
 * description
 *
 * @author shira 2019/05/15 13:45
 */
public interface RabbitMsgService {

	void sendMsg(String msg);

	void sendUser(User user);

}

```





（2）RabbitMsgServiceImpl

```java
package com.ray.study.springboot08messagingrabbitmq.service.impl;


import com.ray.study.springboot08messagingrabbitmq.service.RabbitMsgService;
import com.ray.study.springboot08messagingrabbitmq.entity.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

/**
 * description
 *
 * @author shira 2019/05/15 13:49
 */
@Slf4j
@Service
public class RabbitMsgServiceImpl implements RabbitTemplate.ConfirmCallback, RabbitMsgService {


	@Value("${rabbitmq.queue.msg}")
	private String msgRouting;

	@Value("${rabbitmq.queue.user}")
	private String userRouting;

	@Autowired
	private RabbitTemplate rabbitTemplate;



	@Override
	public void sendMsg(String msg) {
		log.info("发送消息：{}", msg);
		// 设置回调
		rabbitTemplate.setConfirmCallback(this);
		// 发送消息，通过	msgRouting 确定队列
		rabbitTemplate.convertAndSend(msgRouting, msg);
	}

	@Override
	public void sendUser(User user) {
		log.info("发送用户消息：{}", user);
		// 设置回调
		rabbitTemplate.setConfirmCallback(this);
		rabbitTemplate.convertAndSend(userRouting, user);
	}


	/**
	 *  回调确认方法
	 *   spring.rabbitmq.publisher-confirms = true 时，才会进行回调确认，也就是会执行回调方法
	 *   spring.rabbitmq.publisher-confirms = false 时，不管有没有设置回调方法，都不会进行回调确认
	 *
	 * @param correlationData
	 * @param ack
	 * @param cause
	 */
	@Override
	public void confirm(CorrelationData correlationData, boolean ack, String cause) {
		if(ack){
			log.info("消息消费成功");
		}else{
			log.info("消息消费失败：{}"+cause);
		}
	}

}

```





## 6.消息消费者

- RabbitMsgReceiver

```java
package com.ray.study.springboot08messagingrabbitmq.receiver;

import com.rabbitmq.client.Channel;
import com.ray.study.springboot08messagingrabbitmq.entity.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * description
 *
 * @author shira 2019/05/15 14:11
 */
@Component
@Slf4j
public class RabbitMsgReceiver {

	/**
	 * 定义监听字符串队列名称
	 * @param msg
	 */
	@RabbitListener(queues = {"${rabbitmq.queue.msg}"})
	public void reveiveMsg(String msg){
		log.info("收到消息：{}",msg);
	}


	/**
	 * 定义监听用户消息队列名称
	 * @param user
	 */
	@RabbitListener(queues = {"${rabbitmq.queue.user}"})
	public void reveiveUser(User user, Message message, Channel channel){
		log.info("收到用户消息：{}",user);
	}

}

```









## 7.单元测试

- RabbitMsgServiceTest

```java
package com.ray.study.springboot08messagingrabbitmq.service;

import com.ray.study.springboot08messagingrabbitmq.entity.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * description
 *
 * @author shira 2019/05/15 14:19
 */

@RunWith(SpringRunner.class)
@SpringBootTest
public class RabbitMsgServiceTest {

	@Autowired
	RabbitMsgService rabbitMsgService;

	@Test
	public void testRabbit() {
		rabbitMsgService.sendMsg("冷风如刀，以大地为砧板；万里飞雪，将苍穹作烘炉");

		User user = new User();
		user.setId(1L);
		user.setName("阿飞");
		user.setName("18");

		rabbitMsgService.sendUser(user);
	}


}
```





# 参考资料

1. [杨开振__《深入浅出SpringBoot2.x》](https://item.jd.com/12403128.html)
2. [梁桂钊__Spring Boot 揭秘与实战（六） 消息队列篇 - RabbitMQ](http://blog.720ui.com/2017/springboot_06_mq_rabbitmq/)
3. [唐亚峰__一起来学SpringBoot | 第十二篇：初探RabbitMQ消息队列](https://blog.battcn.com/2018/05/22/springboot/v2-queue-rabbitmq/)
4. 

