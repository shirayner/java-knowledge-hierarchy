[TOC]



# 前言

## 1.MongoDB简介

MongoDB使用C++语言编写的非关系型数据库。特点是高性能、易部署、易使用，存储数据十分方便

具体简介请参见：[**mongoDB简介及关键特性**](https://blog.csdn.net/leshami/article/details/52701224)





# 一、SpringBoot 整合 MongoDB



## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringBoot_01_入门示例](./SpringBoot_01_入门示例.md)

```properties
group = 'com.ray.study'
artifact ='spring-boot-06-nosql-mongodb'
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
include 'spring-boot-06-nosql-redis'
include 'spring-boot-06-nosql-mongodb'
```



这样，子工程`spring-boot-05-redis`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
		implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```



### 2.2 引入`MongoDB`依赖

将子模块`spring-boot-05-redis` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
}

```



## 3.修改`application.yml`



```yml
server:
  port: 8088
  servlet:
    context-path: /

spring:
  data:
    mongodb:
      uri: mongodb://root:root@localhost:27017/integrate-mongodb

```





## 4.数据库准备

（1）MongoDB安装及配置

参见：[MongoDB_01_安装及配置](../MongoDB/MongoDB_01_安装及配置.md)



（2）MongoDB 客户端连接工具

参见：[MongoDB_02_客户端连接工具](../MongoDB_02_客户端连接工具.md)



（3）手动创建 integrate-mongodb 数据库，然后执行如下如下语句创建 user 集合

```
db.createCollection("user");
```











## 5.业务实现

### 5.1 entity

```java
package com.ray.study.springboot06nosqlmongodb.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import java.io.Serializable;
import java.util.Date;

/**
 * description
 *
 * @author shira 2019/05/13 11:24
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Document(collection = "user")
public class User implements Serializable {

	private static final long serialVersionUID = 8655851615465363473L;

	@Id
	private String id;

	private String name;

	private Integer age;

	private Date creationDate;

	private Date lastUpdateDate;


	public User(String name, Integer age) {
		this.name = name;
		this.age = age;
		this.creationDate = new Date();
		this.lastUpdateDate = new Date();
	}
}
```



### 5.2 Repositroy

`Spring Data repository ` 抽象统一且简化了数据访问的样板代码，通过`Spring Data repository ` ，我们可以很方便地进行`MongoDB`的访问，在下文会发现我们访问 `MongoDB`和使用`JPA`的过程一模一样，这就是`Spring Data repository`抽象的统一数据访问的魅力了。



```java
package com.ray.study.springboot06nosqlmongodb.repository;

import com.ray.study.springboot06nosqlmongodb.entity.User;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * description
 *
 * @author shira 2019/04/26 10:15
 */
@Repository
public interface UserRepository  extends MongoRepository<User,String> {

	/**
	 * 查询年龄大于等于指定年龄的用户
	 * @param age 年龄
	 * @return userList
	 */
	List<User> findByAgeGreaterThanEqualOrderById(Integer age);

	/**
	 * 根据用户名查询用户信息
	 *
	 * @param name 用户名
	 * @return userList
	 */
	List<User> findAllByName(String name);


	/**
	 * 根据用户名模糊查询
	 * @param name username
	 * @return userList
	 */
	List<User> findByNameLike(String name);

	/**
	 * 模糊查询
	 * @param name
	 * @return
	 */
	List<User> findByNameContaining(String name);
}

```







## 6.单元测试

```java
package com.ray.study.springboot06nosqlmongodb.repository;


import com.ray.study.springboot06nosqlmongodb.entity.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

import static org.hamcrest.CoreMatchers.*;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.is;

/**
 * description
 *
 * @author shira 2019/05/08 17:54
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserRepositoryTest {

	private static final Logger log = LoggerFactory.getLogger(UserRepositoryTest.class);

	@Autowired
	UserRepository userRepository;


	/**
	 * 查询用户
	 */
	@Test
	public void queryUser(){
		// 先清空数据
		userRepository.deleteAll();

		List<User> userToAddList = new ArrayList<>();
		userToAddList.add(new User("aatomcat000",20));
		userToAddList.add(new User("tomcat",21));
		userToAddList.add(new User("tom",21));
		userToAddList.add(new User("tttomcat",22));
		userToAddList.add(new User("tttomcat111",23));
		userToAddList.add(new User("tomcataaa",24));

		// 新增用户
		userToAddList.forEach(user -> {
			userRepository.save(user);
		});


		// 查询所有名字为tom的用户
		List<User> userList = userRepository.findAllByName("tom");
		assertThat(userList.size(), is(1));
		log.info("[findAllByName:tom]:  {}",userList);

		// 查询所有年龄大于等于 21 的用户
		List<User> userList2 = userRepository.findByAgeGreaterThanEqualOrderById(21);
		assertThat(userList2.size(), is(5));
		log.info("[findByAgeGreaterThanEqualOrderById:21]:  {}",userList2);

		// 精确查询，这里的like 并不是模糊查询
		List<User> userList3 = userRepository.findByNameLike("tom");
		// TODO 验证 Spring Data JPA  Like 的查询逻辑
		// assertThat(userList3.size(), is(not(equalTo(6)))); // 等于6？确实是等于6
		log.info("[findByNameLike:tom:size()]:  {}",userList3.size());
		log.info("[findByNameLike:tom]:  {}",userList3);

		// 模糊查询: Containing才是模糊查询
		List<User> userList4 = userRepository.findByNameContaining("tom");
		assertThat(userList4.size(), is(6));
		log.info("[findByNameContaining:tom]:  {}",userList4);
	}

```



## 7.相关异常

### 7.1 UncategorizedMongoDbException: Exception authenticating MongoCredential

#### 7.1.1 异常信息

当MonggoDB配置为如下形式时，会出现如下异常：

```yml
spring:
  data:
    mongodb:
      uri: mongodb://root:root@localhost:27017/integrate-mongodb
```



#### 7.1.2 异常原因

SpringBoot2.x MongoDB认证规则：

> - 当uri地址中既配置了 用户名、密码，又配置了database 时，mongodb 会从所配置的数据库（如示例中的integrate-mongodb）中去进行用户认证。
> - 当uri地址中配置了 用户名、密码，但没有配置了database 时，mongodb 会默认从 admin 数据库中去进行用户认证



上述uri地址中既配置了 用户名、密码，又配置了database，此时 mongodb 默认会从所配置的数据库（integrate-mongodb）中去进行用户认证，显然会认证失败。

因为用户认证信息都在 `admin` 数据库中



#### 7.1.3 异常解决

改成如下配置形式即可。

```yml
spring:
  data:
    mongodb:
      uri: mongodb://root:root@localhost:27017
      database: integrate-mongodb
```















# 二、MongoDB相关知识

参见MongoDB分类

> - [MongoDB总结](../MongoDB/)









# 参考资料

1. [梁桂钊__Spring Boot 揭秘与实战（二） 数据存储篇 - MongoDB](http://blog.720ui.com/2016/springboot_02_data_mongodb/)
2. [Spring Data MongoDB - Reference Documentation](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/)
3. [**mongoDB简介及关键特性**](https://blog.csdn.net/leshami/article/details/52701224)

