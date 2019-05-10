[TOC]



# 前言



# 一、SpringBoot 整合 Mybatis

## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringBoot_01_入门示例](./SpringBoot_01_入门示例.md)

```properties
group = 'com.ray.study'
artifact ='spring-boot-05-mybatis'
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
```



这样，子工程`spring-boot-05-mybatis`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
		implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```



### 2.2 引入`Mybatis`依赖

将子模块`spring-boot-05-mybatis` 的`build.gradle`修改为如下内容：

```groovy
dependencies {

    // mybatis-spring-boot-starter
    implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.0.1'

    // mysql 驱动
    implementation 'mysql:mysql-connector-java'
}
```



## 3.数据库准备

创建数据库`integrate-mybatis`，然后创建`user`表，建表语句如下：

```sql
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user`  (
    `id` bigint(12) NOT NULL AUTO_INCREMENT COMMENT '主键自增',
    `name` varchar(50) NOT NULL COMMENT '用户名',
    `age` int (3) unsigned DEFAULT 3 COMMENT '年龄',
    `creation_date` datetime(0) NOT NULL DEFAULT CURRENT_TIMESTAMP  COMMENT '创建日期',
    `last_update_date` datetime(0) NOT NULL DEFAULT CURRENT_TIMESTAMP  COMMENT '上次更新日期',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户表';
```



## 4.修改`application.yml`

主要配置下数据源与`mybatis`扫描包



```yml
server:
  port: 8088
  servlet:
    context-path: /

spring:
  datasource:       # 配置数据源
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/integrate-mybatis?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
    username: root
    password: root

mybatis:
  type-aliases-package: com.ray.study.springboot05mybatis.mapper
  mapper-locations: classpath:mapper/*.xml
  configuration:
    map-underscore-to-camel-case: true  # 驼峰命名规范 如：数据库字段是  order_id 那么 实体字段就要写成 orderId


```



Mybatis的可选配置项可见`mybatis-spring-boot-autoconfigure`工程下的`spring-configuration-metadata.json`文件

Mybatis的配置属性类为：`MybatisProperties`



## 5.业务实现

### 5.1 entity

```java
package com.ray.study.springboot05mybatis.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;
import java.util.Date;

/**
 * description
 *
 * @author shira 2019/05/09 21:26
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {

	private static final long serialVersionUID = 8655851615465363473L;

	private Long id;

	private String name;

	private Integer age;

	private Date creationDate;

	private Date lastUpdateDate;

}

```





### 5.1 mapper

```java
package com.ray.study.springboot05mybatis.mapper;

import com.ray.study.springboot05mybatis.entity.User;
import org.apache.ibatis.annotations.*;

import java.util.List;
import java.util.Map;

/**
 * description
 *
 * @author shira 2019/05/09 21:27
 */
@Mapper
public interface UserMapper {

	/**
	 * 根据用户名查询用户结果集
	 *
	 * @param name 用户名
	 * @return 查询结果
	 */
	@Select("SELECT * FROM  user  WHERE name = #{name}")
	User findByName(@Param("name") String name);


	/**
	 * 查询所有用户
	 * @return
	 */
	@Results({
			@Result(property = "name", column = "name"),
			@Result(property = "age", column = "age")
	})
	@Select("SELECT name, age FROM user")
	List<User> findAll();

	/**
	 * 保存用户信息
	 *
	 * @param user 用户信息
	 * @return 成功 1 失败 0
	 */
	int insert(User user);

	@Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name}, #{age})")
	int insertByNameAndAge(@Param("name") String name, @Param("age") Integer age);

	@Insert("INSERT INTO USER(NAME, AGE, CREATION_DATE, LAST_UPDATE_DATE) VALUES(#{name}, #{age}, #{creationDate}, #{lastUpdateDate} )")
	int insertByUser(User user);

	@Insert("INSERT INTO USER(NAME, AGE) VALUES(#{name,jdbcType=VARCHAR}, #{age,jdbcType=INTEGER})")
	int insertByMap(Map<String, Object> map);


	@Update("UPDATE user SET age=#{age} WHERE name=#{name}")
	void update(User user);

	@Delete("DELETE FROM user WHERE id =#{id}")
	void delete(Long id);

}
```



## 6.单元测试

- UserMapperTest

```java
package com.ray.study.springboot05mybatis.mapper;

import com.ray.study.springboot05mybatis.entity.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Transactional;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.CoreMatchers.nullValue;
import static org.hamcrest.MatcherAssert.assertThat;

/**
 * description
 *
 * @author shira 2019/05/09 22:01
 */

@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
@Rollback
public class UserMapperTest {

	@Autowired
	private UserMapper userMapper;


	@Test
	public void testUserMapper() {
		// 1.insert:  insert一条数据，并select出来验证
		userMapper.insertByNameAndAge("tom", 21);
		User u = userMapper.findByName("tom");
		assertThat(u.getAge(), is(21));


		// 2.update:  update一条数据，并select出来验证
		u.setAge(30);
		userMapper.update(u);
		u = userMapper.findByName("tom");
		assertThat(u.getAge(), is(30));


		// 3.delete  删除这条数据，并select验证
		userMapper.delete(u.getId());
		u = userMapper.findByName("tom");
		assertThat(u, is(nullValue()));
	}

	@Test
	public void insert() {
		// 1.insert:  insert一条数据，并select出来验证
		User user = new User();
		user.setName("tom");
		user.setAge(21);
		userMapper.insert(user);

		User u = userMapper.findByName("tom");

		assertThat(u.getAge(), is(21));
	}


	@Test
	public void insertByUser() {
		// 1.insert:  insert一条数据，并select出来验证
		User user = new User();
		user.setName("tom");
		user.setAge(21);
		user.setCreationDate(new Date());
		user.setLastUpdateDate(new Date());
		userMapper.insertByUser(user);

		User u = userMapper.findByName("tom");

		assertThat(u.getAge(), is(21));
	}

	@Test
	public void insertByMap() {
		// 1.insert:  insert一条数据，并select出来验证
		Map<String, Object> map = new HashMap<>();
		map.put("name", "tom");
		map.put("age", 21);
		userMapper.insertByMap(map);

		User u = userMapper.findByName("tom");

		assertThat(u.getAge(), is(21));
	}

}
```





# 二、Mybatis相关知识

参见

> - [Mybatis_总结_00_资源帖](https://www.cnblogs.com/shirui/p/9575732.html)
> - [Mybatis_总结](https://www.cnblogs.com/shirui/category/791105.html)











# 参考资料

1. [Spring Boot(六)：如何优雅的使用 Mybatis](http://www.ityouknow.com/springboot/2016/11/06/spring-boot-mybatis.html)
2. [Spring Boot 揭秘与实战（二） 数据存储篇 - MyBatis整合](http://blog.720ui.com/2016/springboot_02_data_mybatis/)
3. [一起来学SpringBoot | 第七篇：整合Mybatis](https://blog.battcn.com/2018/05/09/springboot/v2-orm-mybatis/)
4. 

