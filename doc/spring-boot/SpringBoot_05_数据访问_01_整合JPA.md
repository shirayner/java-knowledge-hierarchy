[TOC]







# 前言



# 一、SpringBoot 整合Jpa

## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringBoot_01_入门示例](./SpringBoot_01_入门示例.md)

```properties
group = 'com.ray.study'
artifact ='spring-boot-05-jpa'
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
```



这样，子工程`spring-boot-05-jpa`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
		implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```



### 2.2 引入`jpa`依赖

将子模块`spring-boot-05-jpa` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    // jpa 
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    
    // mysql 驱动
    implementation 'mysql:mysql-connector-java'
}
```



## 3.修改配置

### 3.1  修改`application.yml`

主要配置下数据源与JPA

由于我们配置的`ddl-auto = update`，因此不需要创建相关数据表，只需要创建`integrate-jpa`数据库即可

```yml
server:
  port: 8088
  servlet:
    context-path: /

spring:
  datasource:       # 配置数据源
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/integrate-jpa?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
    username: root
    password: root
  jpa:                 # 配置jpa
    database: mysql       # 数据库类型
    show-sql: true        # 打印sql语句
    hibernate:
      ddl-auto: update    # 加载 Hibernate时， 自动更新数据库结构

```



数据源的配置类为`DataSourceProperties`，具体的默认配置:

```java
	// 默认配置见 determineDriverClassName 、 determineUsername、determineUrl、determinePassword
	public DataSourceBuilder<?> initializeDataSourceBuilder() {
		return DataSourceBuilder.create(getClassLoader()).type(getType())
				.driverClassName(determineDriverClassName()).url(determineUrl())
				.username(determineUsername()).password(determinePassword());
	}
```



JPA的配置类为`JpaProperties`、`HibernateProperties`



`Hibernate DDL`配置的含义：

> - **create：** 每次运行程序时，都会重新创建表，故而数据会丢失
> - **create-drop：** 每次运行程序时会先创建表结构，然后待程序结束时清空表
> - **upadte：** 每次运行程序，没有表时会创建表，如果对象发生改变会更新表结构，原有数据不会清空，只会更新（推荐使用）
> - **validate：** 运行程序会校验数据与数据库的字段类型是否相同，**字段不同会报错**





## 4.业务实现

### 4.1 model

（1）BaseDO

```java
package com.ray.study.springboot05jpa.model;


import com.fasterxml.jackson.annotation.JsonIgnore;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.MappedSuperclass;
import java.util.Date;

/**
 * BaseDO
 *
 * @author shira 2019/05/08 17:17
 */
@Data
@NoArgsConstructor
@MappedSuperclass  // 声明子类可继承基类字段
public class BaseDO {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@JsonIgnore
	private Date creationDate;

	@JsonIgnore
	private Date lastUpdateDate;
}


```



（2）User

```java
package com.ray.study.springboot05jpa.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.Table;


/**
 * 用户模型
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "user")
public class User extends BaseDO{

	/**
	 * 用户名
	 */
	private String name;

	/**
	 * 年龄
	 */
	private Integer age;


}


```



### 4.2 repository

- UserRepository

```java
package com.ray.study.springboot05jpa.repository;

import com.ray.study.springboot05jpa.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * description
 *
 * @author shira 2019/04/26 10:15
 */
@Repository
public interface UserRepository extends JpaRepository<User, Long>  {

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





## 5.单元测试

- UserRepositoryTest



```java
package com.ray.study.springboot05jpa.repository;

import com.ray.study.springboot05jpa.model.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

import static org.hamcrest.CoreMatchers.notNullValue;
import static org.hamcrest.CoreMatchers.nullValue;
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

	@Autowired
	UserRepository userRepository;


	/**
	 * 查询用户
	 */
	@Test
	public void queryUser(){
		List<User> userToAddList = new ArrayList<>();
		userToAddList.add(new User("aatomcat000",20));
		userToAddList.add(new User("tomcat",21));
		userToAddList.add(new User("tttomcat",22));
		userToAddList.add(new User("tttomcat111",23));
		userToAddList.add(new User("tomcataaa",24));

		// 新增用户
		userToAddList.forEach(user -> {
			userRepository.save(user);
		});




		// 查询所有名字为tom的用户
		List<User> userList = userRepository.findAllByName("tom");
		// assertThat(userList.size(), is(1));
		System.out.println(userList);

		// 查询所有年龄大于等于 21 的用户
		List<User> userList2 = userRepository.findByAgeGreaterThanEqualOrderById(21);
		//assertThat(userList2.size(), is(notNullValue()));
		System.out.println(userList2);

		// 精确查询，这里的like 并不是模糊查询
		List<User> userList3 = userRepository.findByNameLike("tom");
		//assertThat(userList3.size(), is(notNullValue()));
		System.out.println(userList3);

		// 模糊查询: Containing才是模糊查询
		List<User> userList4 = userRepository.findByNameContaining("tom");
		//assertThat(userList4.size(), is(notNullValue()));
		System.out.println(userList4);
	}


	/**
	 * 新增用户
	 */
	@Test
	public void insertUser(){

		User user = new User();
		user.setName("tom");
		user.setAge(21);
		User user1 = userRepository.save(user);

		assertThat(user.getId(),is(notNullValue()));
	}


	/**
	 * 更新用户
	 */
	@Test
	public void updateUser(){

		User user = new User();
		user.setId(1L);
		user.setName("tom2");
		user.setAge(1212);
		User user1 = userRepository.save(user);

		Optional<User> optionalUser = userRepository.findById(1L);
		User updatedUser = null;
		if(optionalUser.isPresent()){
			updatedUser = optionalUser.get();
		}

		assertThat(updatedUser.getAge(),is(1212));
	}


	/**
	 * 删除用户
	 */
	@Test
	public void deleteUser(){
		User user = new User();
		user.setName("tom2");
		userRepository.delete(user);

		List<User> userList = userRepository.findAllByName("tom2");
		assertThat(userList, is(nullValue()));

	}

}
```







# 二、JPA相关知识

参见 JPA分类（未完待续）







# 参考资料

1. [纯洁的微笑__Spring Boot(五)：Spring Boot Jpa 的使用](http://www.ityouknow.com/springboot/2016/08/20/spring-boot-jpa.html)
2. [唐亚峰__一起来学SpringBoot | 第六篇：整合SpringDataJpa](https://blog.battcn.com/2018/05/08/springboot/v2-orm-jpa/)
3. 











