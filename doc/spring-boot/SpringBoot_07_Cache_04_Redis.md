[TOC]



# 前言



在  [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md)  这一节中，我们了解了 Spring 声明式缓存抽象，知道了：

> - 我们使用Spring声明式缓存，使用的是其抽象，无关底层实现。也就是说，不管你底层使用的是什么缓存技术，对Spring声明式缓存的使用是一致的
> - 若想要采用不同的缓存技术，只需要替换掉底层注册的 CacheManager 即可



那么，这一节我们将采用 `redis`作为缓存技术，来实现一个Spring声明式缓存的实例，这显然就很简单了，一共两步：

> - 引入 `redis`依赖
> - 开启缓存支持
> - 配置 redis
> - 使用声明式缓存

其他步骤同上一步： [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md) 



# 一、SpringBoot 缓存整合 redis

## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringBoot_01_入门示例](./SpringBoot_01_入门示例.md)

```properties
group = 'com.ray.study'
artifact ='spring-boot-07-cache-redis'
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
```



这样，子工程`spring-boot-07-cache-redis`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
		implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```



### 2.2 引入`redis`依赖

将子模块`spring-boot-07-cache-redis` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    // spring-boot-starter-cache
    implementation 'org.springframework.boot:spring-boot-starter-cache'

    // redis 依赖 和 common pool2 ，要使用redis连接池就需要引入此common pool2
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'org.apache.commons:commons-pool2'


    // mysql 驱动和 jpa
    implementation 'mysql:mysql-connector-java'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

}
```



## 3.修改配置

### 3.1 修改`application.yml`

关于 redis cache 的可以使用默认配置：

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





### 3.2 CacheConfiguration

开启缓存支持

> 通常SpringBoot默认的keyGenerator 是SimpleKeyGenerator，这个策略是以参数作为key值，如果参数为空的，就会返回SimpleKey[]字符串，这对于很多无参的方法的就有问题了.
> 向Spring容器中重新注册一个实现了 `org.springframework.cache.interceptor.keyGenerator` 这个接口的Bean即可，将key值设置为类名+方法名+参数名，这样就不会冲突了



```java
package com.ray.study.springboot07cache.caffeine.config;

import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


/**
 * Spring cache 配置类
 *
 * @author shira 2019/05/13 18:45
 */
@Configuration
@EnableCaching
public class CacheConfiguration {

	@Bean
	public KeyGenerator caffeineKeyGenerator() {
		return (target, method, params) -> {  // KeyGenerator#generate
			StringBuilder sb = new StringBuilder();
			sb.append(target.getClass().getName());
			sb.append(method.getName());
			for (Object obj : params) {
				sb.append(obj.toString());
			}
			return sb.toString();
		};
	}



}
```



## 3.其他部分

其他部分（数据库准备、业务实现、单元测试）参加： [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md) 

不过请注意**要缓存的JavaBean必须实现Serializable接口，因为Spring会将对象先序列化再存入 Redis**，因此

```java
@Getter
@Setter
@NoArgsConstructor
@MappedSuperclass 
public class BaseDO implements Serializable {   // BaseDO需要实现Serializable接口，否则会抛无法序列化的异常

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;

	@JsonIgnore
	private Date creationDate;

	@JsonIgnore
	private Date lastUpdateDate;
}


```









# 参考资料

1. [springboot 2 集成 redis 缓存](https://juejin.im/entry/5b433c3de51d45191716e01c)
2. [Spring Boot缓存实战 Caffeine](https://www.jianshu.com/p/c72fb0c787fc)

