[TOC]

# 前言

这一节的内容大部分来自官方文档：

> - [SpringBoot 官方文档中文版@24.外部配置](https://springcloud.cc/spring-boot.html)
> - [SpringBoot Reference Document @24. Externalized Configuration](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config)





# 一、读取应用配置的方式

这部分有源码：源码地址：

> https://github.com/shirayner/spring-boot-seeds/tree/master/spring-boot-10-config-readconfigproperty



## 1.Environment

关于Spring Environment 抽象，可参考：

> - [Spring 环境抽象 Environment](https://blog.csdn.net/andy_zhang2007/article/details/78511815)
> - [Spring Environment Abstraction](https://www.javazhiyin.com/13242.html)
> - [spring 4.3.8 教程翻译：7.13 抽象环境 7.14 注册 LoadTimeWeaver](http://bbs.bubbt.com/symphony/article/1495871278924?p=1&m=0)
> - [Spring Environment（一）API 使用](https://www.cnblogs.com/binarylei/p/10280374.html)
> - [Spring Boot 多环境配置最佳实践](https://www.infoq.cn/article/Q-ese4CxV2IWmltsJcGX)



简单来说，Environment 是一个通用的读取应用程序运行时的环境变量的类，可以读取 application.properties 、命令行输入参数、系统属性、操作系统环境变量等。



读取属性示例：

```java
@Component
public class EnvironmentService {

    @Autowired
    Environment env;

    public void printEnv() {
       // 读取程序运行目录，来自系统属性
       System.out.println(env.getProperty("user.dir"));  
       
       // 读取用户home目录，来自系统属性
       System.out.println(env.getProperty("user.home")); 
      
       // 读取JAVA_HOME这个环境变量，来自系统环境变量
       System.out.println(env.getProperty("JAVA_HOME")); 
      
       //读取server.port，来自application.properties 
       System.out.println(env.getProperty("server.port",Integer.class)); 
    }
}
```



> Environment  是SpringBoot 最早初始化的一个类，因此可以用在Spring应用的任何地方



## 2.@Value

- 通过 @Value 可以将配置文件中配置的属性注入到修饰的字段上
-  @Value 支持SpEL 表达式，并且如果属性不存在，可以为其提供一个默认值

例如：

（1）application.yml  配置如下：

```yml
server:
  port: 8088
  servlet:
    context-path: /


csdn:
  blog:
    name: shirayner的博客
    homePage: https://blog.csdn.net/qq_26981333
    articles:
      - author: shirayner
        title: SpringBoot_00_1_资源帖
        url: https://blog.csdn.net/qq_26981333/article/details/89949800
      - author: shirayner
        title: SpringBoot_01_入门示例
        url: https://blog.csdn.net/qq_26981333/article/details/89949895

```



（2）测试类如下:

```java
package com.ray.study.springboot10configreadconfigproperty.config;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

/**
 * description
 *
 * @author shira 2019/05/16 21:37
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class ValueTest {

	@Value("${csdn.blog.name:ray}")  //如果配置属性不存在，则给出默认值 ray 
	private String blogName;

	@Value("${csdn.blog.homePage}")
	private String homePage;

	@Test
	public void testGetValue(){
		assertThat(blogName,is("shirayner的博客"));
		assertThat(homePage,is("https://blog.csdn.net/qq_26981333"));
	}


}

```



> @Value 并不能在任何Spring 管理的Bean 中使用，因为＠Value 本身是通过
> AutowiredAnnotationBeanPostProcessor 实现的，它是 BeanPostProcessor 接口的实现类，
> 因此任何 BeanPostProcessor 和 BeanFactoryPost Processor 的子类中都不能使用 ＠Value 来
> 注入属性，因为那时候 ＠Value 还没有被处理。



## 3.ConfigurationProperties

### 3.1 读取 application.yml 配置

通常情况下，将一组同样类型的配置属性映射为一个类更为方便。

示例如下：

（1）配置文件：还是采用前面使用的 application.yml 

```yml
server:
  port: 8088
  servlet:
    context-path: /


csdn:
  blog:
    name: shirayner的博客
    homePage: https://blog.csdn.net/qq_26981333
    articles:
      - author: shirayner
        title: SpringBoot_00_1_资源帖
        url: https://blog.csdn.net/qq_26981333/article/details/89949800
      - author: shirayner
        title: SpringBoot_01_入门示例
        url: https://blog.csdn.net/qq_26981333/article/details/89949895


```



（2）配置类 

- BlogProperties

    > - 属性映射规则： 配置文件中配置属性的"-"和"_"会被去掉并转成驼峰形式
    > - 被映射属性需要提供getter/setter，而@Value却不需要

```java
package com.ray.study.springboot10configreadconfigproperty.config;

import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * 配置属性类
 *  （1）属性映射规则： 配置文件中配置属性的"-"和"_"会被去掉并转成驼峰形式
 *   也就是说配置文件中的 homePage 可以写成： home-page/home_page/homePage
 *
 *	（2）被映射属性需要提供getter/setter，而@Value却不需要
 *
 * @author shira 2019/05/16 21:18
 */
@Data
@NoArgsConstructor
@Component
@ConfigurationProperties(prefix = "csdn.blog")
public class BlogProperties {

	private String name;

	private String homePage;

	private List<Article> articles;

}

```



- Article

```java
package com.ray.study.springboot10configreadconfigproperty.config;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

/**
 * description
 *
 * @author shira 2019/05/16 21:19
 */
@Getter
@Setter
@NoArgsConstructor
public class Article {

	private String author;

	private String title;

}

```



（3）测试类如下

```java
package com.ray.study.springboot10configreadconfigproperty.config;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

/**
 * description
 *
 * @author shira 2019/05/16 21:32
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class ConfigurationPropertiesTest {

	@Autowired
	BlogProperties blogProperties;

	@Test
	public void testGetConfigurationProperties() {
		assertThat(blogProperties.getName(),is("shirayner的博客"));
		assertThat(blogProperties.getArticles().size(),is(2));
	}

}
```





### 3.2 读取自定义properties配置文件

除了将配置属性配置在 application-*.yml 或 application.yml （或.properties文件），还可以将配置属性挪到另一个properties 配置文件中。

（1）新建 user.properties，内容如下：

```properties
csdn.user.nike-name=shirayner
csdn.user.age=25
csdn.user.email=1766211120@qq.com
```



（2）UserProperties

```java
package com.ray.study.springboot10configreadconfigproperty.config;

import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

/**
 * 配置属性映射类
 * 注意：@PropertySource 默认只能加载 properties 配置文件，而无法加载 yml 配置文件
 *
 * @author shira 2019/05/16 21:18
 */
@Data
@NoArgsConstructor
@Component
@PropertySource("classpath:user.properties")
@ConfigurationProperties(prefix = "csdn.user")
public class UserProperties {

	private String nikeName;

	private Long age;

	private String email;

}

```



（3）测试类

```java
package com.ray.study.springboot10configreadconfigproperty.config;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

/**
 * description
 *
 * @author shira 2019/05/16 21:48
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class PropertySourceTest {

	@Autowired
	UserProperties userProperties;

	@Test
	public void testGetProperty(){
		assertThat(userProperties.getNikeName(), is("shirayner"));
		assertThat(userProperties.getAge(), is(25L));
		assertThat(userProperties.getEmail(), is("1766211120@qq.com"));
	}
}

```



注意：@PropertySource 默认只能加载 properties 配置文件，而无法加载 yml 配置文件；要实现yml配置文件的加载，需要指定一个自定义的yaml属性加载工厂，参见下一小节



### 3.3 读取自定义yml配置文件

这篇博客 [@PropertySource 注解实现读取 yml 文件](https://juejin.im/post/5c4aea2e51882522c03ea475) 从源码上讲解了为什么无法加载yml配置文件，可先阅读此博客，再来实现下面的例子。

这里将在上一小节 读取自定义properties配置文件 的基础上，将其修改为能够读取yml配置文件的

（1）新建 user-default.yml 文件，内容与 user.properties 文件内容保持一致

```yml
csdn:
  user:
    nike-name: shirayner
    age: 25
    email: 1766211120@qq.com
```



（2）自定义yml属性加载工厂

```java
package com.ray.study.springboot10configreadconfigproperty.config;

import org.springframework.boot.env.YamlPropertySourceLoader;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.support.DefaultPropertySourceFactory;
import org.springframework.core.io.support.EncodedResource;
import java.io.IOException;

/**
 * 实现yaml配置文件加载工厂，以使用@PropertySource注解加载指定yaml文件的配置
 *
 * @author shira 2019/05/16 23:25
 */
public class YamlPropertyLoaderFactory extends DefaultPropertySourceFactory {
	@Override
	public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {

		if (null == resource) {
			super.createPropertySource(name, resource);
		}
		return new YamlPropertySourceLoader().load(resource.getResource().getFilename(), resource.getResource()).get(0);
	}
}

```



（3）@PropertySource 指定属性加载工厂为 YamlPropertyLoaderFactory 

```java
package com.ray.study.springboot10configreadconfigproperty.config;

import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

/**
 * 配置属性映射类
 * 注意：@PropertySource 只能加载 properties 配置文件，而无法加载 yml 配置文件
 *
 * @author shira 2019/05/16 21:18
 */
@Data
@NoArgsConstructor
@Component
//@PropertySource("classpath:user.properties")
@PropertySource(value = "classpath:user-default.yml", factory = YamlPropertyLoaderFactory.class)
@ConfigurationProperties(prefix = "csdn.user")
public class UserProperties {

	private String nikeName;

	private Long age;

	private String email;

}

```



（4）再次运行前面的测试类，会发现这次测试通过了



# 二、外部化配置

> 这一节的内容来自官方文档：[[24. Externalized Configuration](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config)]，可参见中文版：[https://springcloud.cc/spring-boot.html]

Spring Boot允许您外部化您的配置，以便您可以在不同的环境中使用相同的应用程序代码。

您可以使用属性文件，YAML文件，环境变量和命令行参数来外部化配置。

Property值可以通过使用`@Value`注释直接注入beans，通过Spring的`Environment`抽象访问，或 通过`@ConfigurationProperties` [绑定到结构化对象](https://springcloud.cc/spring-boot.html#boot-features-external-config-typesafe-configuration-properties)。

## 1.属性加载优先级

Spring Boot使用非常特殊的`PropertySource`顺序，旨在允许合理地覆盖值。按以下顺序考虑属性：

> 1. [Devtools](https://springcloud.cc/spring-boot.html#using-boot-devtools-globalsettings) 主目录上的[全局设置属性](https://springcloud.cc/spring-boot.html#using-boot-devtools-globalsettings)（当devtools处于活动状态时`~/.spring-boot-devtools.properties`）
> 2. [`@TestPropertySource`](https://docs.spring.io/spring/docs/5.1.3.RELEASE/javadoc-api/org/springframework/test/context/TestPropertySource.html) 测试上的注释
> 3. `properties`属于您的测试。可 [用于测试特定应用程序片段](https://springcloud.cc/spring-boot.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)[`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/api/org/springframework/boot/test/context/SpringBootTest.html)的 [测试注释](https://springcloud.cc/spring-boot.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)
> 4. 命令行参数
> 5. 来自`SPRING_APPLICATION_JSON`的属性（嵌入在环境变量或系统属性中的内联JSON）
> 6. `ServletConfig` init参数
> 7. `ServletContext` init参数
> 8. JNDI来自`java:comp/env`
> 9. Java系统属性（`System.getProperties()`）
> 10. OS环境变量
> 11. `RandomValuePropertySource`仅在`random.*`中具有属性
> 12. Jar包外部的（`application-{profile}.properties`和YAML变体）
> 13. Jar包内部的（`application-{profile}.properties`和YAML变体）
> 14. Jar包外部的（`application.properties`和YAML变体）
> 15. Jar包内部的（`application.properties`和YAML变体）
> 16. `@Configuration`注解类上的`@PropertySource`
> 17. 默认属性（通过设置`SpringApplication.setDefaultProperties`指定）









## 2.配置文件加载优先级

`SpringApplication`从以下位置的`application.properties`文件加载属性，并将它们添加到Spring `Environment`：

> 1. 当前目录的`/config`子目录
> 2. 当前目录
> 3. classpath `/config`包
> 4. 类路径根

列表按优先级排序（在列表中较高位置定义的属性将覆盖在较低位置中定义的属性）。



如果您不喜欢`application.properties`作为配置文件名，则可以通过指定`spring.config.name`环境属性来切换到另一个文件名。您还可以使用`spring.config.location`环境属性（以逗号分隔的目录位置或文件路径列表）来引用显式位置。以下示例显示如何指定其他文件名：

```bash
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```



如果`spring.config.location`包含目录（而不是文件），则它们应以`/`结束（并且在运行时，在加载之前附加从`spring.config.name`生成的名称，包括特定于配置文件（application-{profile}.properties）的文件名） 。

`spring.config.location`中指定的文件按原样使用，不支持特定于配置文件的变体，并且被任何特定于配置文件的属性覆盖。



以相反的顺序搜索配置位置。默认情况下，配置的位置为`classpath:/,classpath:/config/,file:./,file:./config/`。生成的搜索顺序如下：

> 1. `file:./config/`
> 2. `file:./`
> 3. `classpath:/config/`
> 4. `classpath:/`



使用`spring.config.location`配置自定义配置位置时，它们会替换默认位置。例如，如果`spring.config.location`配置了值`classpath:/custom-config/,file:./custom-config/`，则搜索顺序将变为：

> 1. `file:./custom-config/`
> 2. `classpath:custom-config/`



或者，当使用`spring.config.additional-location`配置自定义配置位置时，除了默认位置外，还会使用它们。在默认位置之前搜索其他位置。例如，如果配置了`classpath:/custom-config/,file:./custom-config/`的其他位置，则搜索顺序将变为以下内容：

> 1. `file:./custom-config/`
> 2. `classpath:custom-config/`
> 3. `file:./config/`
> 4. `file:./`
> 5. `classpath:/config/`
> 6. `classpath:/`



此搜索顺序允许您在一个配置文件中指定默认值，然后有选择地覆盖另一个配置文件中的值。您可以在`application.properties`（或您使用`spring.config.name`选择的任何其他基本名称）中的某个默认位置为您的应用程序提供默认值。然后，可以在运行时使用位于其中一个自定义位置的不同文件覆盖这些默认值。



## 3. 命令行参数

```bash
 java -jar myproject.jar --server.port=8088
```



`random.int*`语法为`OPEN value (,max) CLOSE`，其中`OPEN,CLOSE`为任意字符，`value,max`为整数。如果提供`max`，那么`value`是最小值，`max`是最大值（不包括）。



## 4. 配置随机值

`RandomValuePropertySource`对于注入随机值（例如，进入秘密或测试用例）非常有用。它可以生成整数，长整数，uuids或字符串，如以下示例所示：

```
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```













# 参考资料

1. [李家智__《**Spring Boot 2精髓：从构建小系统到架构分布式大系统**》](https://item.jd.com/12214143.html)

2. [杨开振__《**深入浅出Spring Boot 2.x**》](https://item.jd.com/12403128.html)

3. [疯狂软件__《**Spring Boot 2企业应用实战**》](https://item.jd.com/12373402.html)

4. [@PropertySource 注解实现读取 yml 文件](https://juejin.im/post/5c4aea2e51882522c03ea475)

5. [SpringBoot工程 yaml配置文件映射到pojo对象](https://jacheng.top/2018/11/01/SpringBoot%E5%B7%A5%E7%A8%8B-yaml%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E6%98%A0%E5%B0%84%E5%88%B0pojo%E5%AF%B9%E8%B1%A1/)

6. [SpringBoot中文文档](https://springcloud.cc/spring-boot.html)

    