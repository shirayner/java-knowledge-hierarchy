[TOC]



# 前言





# 一、创建父工程

## 1.项目坐标

```properties
group = 'com.ray.study.dockercompose'
artifact ='spring-cloud-docker-compose-gradle'
```





# 二、创建注册中心

## 1.项目坐标

```
group = 'com.ray.study.dockercompose'
artifact ='dc-register-server-eureka'
```



## 2.引入依赖

（1）修改父工程的 `settings.gradle`

```groovy
pluginManagement {
    repositories {
        gradlePluginPortal()
    }
}
rootProject.name = 'spring-cloud-docker-compose-gradle'
include 'register-server-eureka'
```



（2）修改 `register-server-eureka`的 gradle 构建文件

```groovy
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

```



## 3.修改配置

### 3.1 修改 application.yml

```yml
server:
  port: 8761  #服务端口

spring:
  application:
    name: eurka-server   #指定服务名

eureka:
  instance:
    hostname: localhost
  server:
    enable-self-preservation: false  #是否开启自我保护模式
    eviction-interval-timer-in-ms: 60000  #服务注册表清理间隔（单位毫秒，默认是60*1000）
  client:
    registerWithEureka: false  #服务注册，是否将自己注册到Eureka服务注册中心，单机版本时，为false就好
    fetchRegistry: false       #服务发现，是否从Eureka中获取注册信息
    serviceUrl:              #Eureka客户端与Eureka服务端的交互地址，高可用状态配置对方的地址，单机状态配置自己（如果不配置则默认本机8761端口）
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/



```



### 3.1 启用 EurekaServer

```java
package com.ray.study.dockercompose.registerservereureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class RegisterServerEurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(RegisterServerEurekaApplication.class, args);
	}

}

```





# 二、创建Model工程

## 1.项目坐标

```
group = 'com.ray.study.dockercompose'
artifact ='dc-model'
```



## 2.引入依赖







# 三、创建服务提供者

## 1.项目坐标

```
group = 'com.ray.study.dockercompose'
artifact ='dc-product-service'
```



## 2.引入依赖

（1）修改父工程的 `settings.gradle`

```groovy
pluginManagement {
    repositories {
        gradlePluginPortal()
    }
}
rootProject.name = 'spring-cloud-docker-compose-gradle'
include 'register-server-eureka'
include 'provider'
```



（2）修改 `provider`的 gradle 构建文件

```groovy
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // eureka client
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
```



## 3.修改配置

### 3.1 修改 application.yml

```yml
server:
  port: 8761  #服务端口

spring:
  application:
    name: eurka-server   #指定服务名

eureka:
  instance:
    hostname: localhost
  server:
    enable-self-preservation: false  #是否开启自我保护模式
    eviction-interval-timer-in-ms: 60000  #服务注册表清理间隔（单位毫秒，默认是60*1000）
  client:
    registerWithEureka: false  #服务注册，是否将自己注册到Eureka服务注册中心，单机版本时，为false就好
    fetchRegistry: false       #服务发现，是否从Eureka中获取注册信息
    serviceUrl:              #Eureka客户端与Eureka服务端的交互地址，高可用状态配置对方的地址，单机状态配置自己（如果不配置则默认本机8761端口）
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/



```



### 3.1 启用 EurekaServer

```java
package com.ray.study.dockercompose.registerservereureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class RegisterServerEurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(RegisterServerEurekaApplication.class, args);
	}

}

```







# 三、创建服务消费者

## 1.项目坐标

```
group = 'com.ray.study.dockercompose'
artifact ='dc-order-service'
```



## 2.引入依赖

（1）修改父工程的 `settings.gradle`

```groovy
pluginManagement {
    repositories {
        gradlePluginPortal()
    }
}
rootProject.name = 'spring-cloud-docker-compose-gradle'
include 'register-server-eureka'
include 'provider'
```



（2）修改 `provider`的 gradle 构建文件

```groovy
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // eureka client
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
```



## 3.修改配置

### 3.1 修改 application.yml

```yml
server:
  port: 8761  #服务端口

spring:
  application:
    name: eurka-server   #指定服务名

eureka:
  instance:
    hostname: localhost
  server:
    enable-self-preservation: false  #是否开启自我保护模式
    eviction-interval-timer-in-ms: 60000  #服务注册表清理间隔（单位毫秒，默认是60*1000）
  client:
    registerWithEureka: false  #服务注册，是否将自己注册到Eureka服务注册中心，单机版本时，为false就好
    fetchRegistry: false       #服务发现，是否从Eureka中获取注册信息
    serviceUrl:              #Eureka客户端与Eureka服务端的交互地址，高可用状态配置对方的地址，单机状态配置自己（如果不配置则默认本机8761端口）
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/



```



### 3.1 启用 EurekaServer

```java
package com.ray.study.dockercompose.registerservereureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class RegisterServerEurekaApplication {

	public static void main(String[] args) {
		SpringApplication.run(RegisterServerEurekaApplication.class, args);
	}

}

```







# 三、创建服务消费者





# 四、相关异常

## 1.项目依赖正常，却报找不到 model

原因springboot默认只扫描启动类所在包下的Bean，而我们将model分离出去之后，只能就扫描不到了，这是需要手动指定扫描的实体类

```java
@SpringBootApplication
@EntityScan("com.ray.study.dockercompose.dcmodel.entity")//扫描实体类
@EnableDiscoveryClient
public class DcProductServiceApplication {

```



## 2. 实体类含有mysql关键字

这是在jpa创建表示，会报出如下错误：

```
SQLSyntaxErrorException: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near
```

将其改成正确的即可。（原本Order中的 status 字段改为 orderStatus 即可）



## 3. eureka-server中找不到 [templates/]



```
2019-06-05 20:26:03.427 DEBUG 23864 --- [           main] o.s.w.s.v.f.FreeMarkerConfigurer         : Cannot resolve template loader path [classpath:/templates/] to [java.io.File]: using SpringTemplateLoader as fallback

java.io.FileNotFoundException: class path resource [templates/] cannot be resolved to absolute file path because it does not reside in the file system: jar:file:/D:/maven-repository/org/springframework/cloud/spring-cloud-netflix-eureka-server/2.1.0.RELEASE/spring-cloud-netflix-eureka-server-2.1.0.RELEASE.jar!/templates/
	at org.springframework.util.ResourceUtils.getFile(ResourceUtils.java:217) ~[spring-core-5.1.7.RELEASE.jar:5.1.7.RELEASE]
	at org.springframework.core.io.AbstractFileResolvingResource.getFile(AbstractFileResolvingResource.java:154) ~[spring-core-5.1.7.RELEASE.jar:5.1.7.RELEASE]
	at org.springframework.ui.freemarker.FreeMarkerConfigurationFactory.getTemplateLoaderForPath(FreeMarkerConfigurationFactory.java:345) [spring-context-support-5.1.7.RELEASE.jar:5.1.7.RELEASE]
	at org.springframework.ui.freemarker.FreeMarkerConfigurationFactory.createConfiguration(FreeMarkerConfigurationFactory.java:297) [spring-context-support-5.1.7.RELEASE.jar:5.1.7.RELEASE]
```

这是一个debug 日志，不用管

参见：https://github.com/spring-cloud/spring-cloud-netflix/issues/121



## 4.No bean named 'entityManagerFactory' available

将本地仓库中的 Hibernate jar全删掉，然后项目中不要带版本号，如下所示

```
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
        </dependency>
```





## 5. No main manifest attribute in jar Maven and Spring Boot

SpringBoot 运行jar包提示没有主清单属性 no main manifest attribute, in /app.jar

```xml
			 <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <goals>
                                <goal>repackage</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
```

