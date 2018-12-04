[TOC]



# 一、前言

上一节，我们使用springboot 实现了一个简单的请求响应流程。



下面我们尝试使用Spring MVC来实现一组对User对象操作的RESTful API，配合注释详细说明在Spring MVC中如何映射HTTP请求、如何传参、如何编写单元测试。





# 二、实例

## 1. 引入依赖

在父工程的 的build.gradle 中 的subprojects函数中加入 lombok 的依赖

```groovy
buildscript {
    ext {
        springBootVersion = '2.1.1.RELEASE'
    }
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}


//配置所有项目
allprojects {
    //公共插件插件
    apply plugin: 'java'

    //公共属性
    group = 'com.ray.study'
    version = '0.0.1-SNAPSHOT'

    //编译版本
    sourceCompatibility = 1.8
    targetCompatibility = 1.8

}

//构建依赖
subprojects {

    //应用插件
    apply plugin: 'idea'
    apply plugin: 'maven'
    apply plugin: 'org.springframework.boot'
    apply plugin: 'io.spring.dependency-management'

    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
    }

    dependencies {
        compile('org.springframework.boot:spring-boot-starter-web')
        testCompile('org.springframework.boot:spring-boot-starter-test')

        // 4.lombok
        compile('org.projectlombok:lombok')
    }
}

repositories {
    maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }

}


```







## 2.RESTful API设计



| 请求类型 | URL         | 功能说明       |
| -------- | ----------- | -------------- |
| GET      | /users      | 查询用户列表   |
| POST     | /users      | 新增用户       |
| GET      | /users/{id} | 根据id查询用户 |
| PUT      | /users/{id} | 根据id更新用户 |
| DELETE   | /users/{id} | 根据id删除用户 |



## 3.User

实体类如下

```java
@Setter
@Getter
@NoArgsConstructor
public class User {
   private Long id;
   private String name;
   private Integer age;
}
```



## 4.UserController

对应的Controller如下

```java
@RestController
@RequestMapping(value="/users")     // 通过这里配置使下面的映射都在/users下
public class UserController {
   // 创建线程安全的Map
   static Map<Long, User> users = Collections.synchronizedMap(new HashMap<Long, User>());

   @RequestMapping(value="/", method=RequestMethod.GET)
   public List<User> getUserList() {
      // 处理"/users/"的GET请求，用来获取用户列表
      // 还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递
      List<User> r = new ArrayList<User>(users.values());
      return r;
   }

   @RequestMapping(value="/", method=RequestMethod.POST)
   public String postUser(@ModelAttribute User user) {
      // 处理"/users/"的POST请求，用来创建User
      // 除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数
      users.put(user.getId(), user);
      return "success";
   }

   @RequestMapping(value="/{id}", method=RequestMethod.GET)
   public User getUser(@PathVariable Long id) {
      // 处理"/users/{id}"的GET请求，用来获取url中id值的User信息
      // url中的id可通过@PathVariable绑定到函数的参数中
      return users.get(id);
   }

   @RequestMapping(value="/{id}", method= RequestMethod.PUT)
   public String putUser(@PathVariable Long id, @ModelAttribute User user) {
      // 处理"/users/{id}"的PUT请求，用来更新User信息
      User u = users.get(id);
      u.setName(user.getName());
      u.setAge(user.getAge());
      users.put(id, u);
      return "success";
   }

   @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
   public String deleteUser(@PathVariable Long id) {
      // 处理"/users/{id}"的DELETE请求，用来删除User
      users.remove(id);
      return "success";
   }

}
```



## 5.UserControllerTest

UserController对应的测试类如下

```java
package com.ray.study.springboot.web;

import com.ray.study.springboot.ApplicationTests;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.springframework.test.web.servlet.RequestBuilder;

import static org.hamcrest.core.IsEqual.equalTo;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

/**
 * @desc
 *
 * @author shirayner
 * @date 2018/12/4
 */

@Slf4j
public class UserControllerTest extends ApplicationTests {

   RequestBuilder request = null;

   @Test
   public void getUserList() throws Exception {
      // 1、get查一下user列表，应该为空
      request = get("/users/");

      mockMvc.perform(request).andExpect(status().isOk())
            .andExpect(content().string(equalTo("[{\"id\":1,\"name\":\"测试大师\",\"age\":20}]")));
   }

   @Test
   public void postUser() throws Exception {
      // 2、post提交一个user
      request = post("/users/").param("id", "1").param("name", "测试大师").param("age", "20");

      mockMvc.perform(request).andExpect(content().string(equalTo("success")));


   }


   @Test
   public void putUser() throws Exception {
      // 4、put修改id为1的user
      request = put("/users/1").param("name", "测试终极大师").param("age", "30");
      mockMvc.perform(request).andExpect(content().string(equalTo("success")));


   }

   @Test
   public void getUser() throws Exception {
      // 5、get一个id为1的user
      request = get("/users/1");
      mockMvc.perform(request).andExpect(content().string(equalTo("{\"id\":1,\"name\":\"测试终极大师\",\"age\":30}")));

   }


   @Test
   public void deleteUser() throws Exception {
      // 6、del删除id为1的user
      request = delete("/users/1");
      mockMvc.perform(request).andExpect(content().string(equalTo("success")));
   }

   @Test
   public void testAll() throws Exception {
      // 测试UserController

      // 1、get查一下user列表，应该为空
      request = get("/users/");

      mockMvc.perform(request)
            .andExpect(status().isOk())
            .andExpect(content().string(equalTo("[]")));

      // 2、post提交一个user
      request = post("/users/")
            .param("id", "1")
            .param("name", "测试大师")
            .param("age", "20");

      mockMvc.perform(request)
            .andExpect(content().string(equalTo("success")));

      // 3、get获取user列表，应该有刚才插入的数据
      request = get("/users/");
      mockMvc.perform(request)
            .andExpect(status().isOk())
            .andExpect(content().string(equalTo("[{\"id\":1,\"name\":\"测试大师\",\"age\":20}]")));

      // 4、put修改id为1的user
      request = put("/users/1")
            .param("name", "测试终极大师")
            .param("age", "30");
      mockMvc.perform(request)
            .andExpect(content().string(equalTo("success")));

      // 5、get一个id为1的user
      request = get("/users/1");
      mockMvc.perform(request)
            .andExpect(content().string(equalTo("{\"id\":1,\"name\":\"测试终极大师\",\"age\":30}")));

      // 6、del删除id为1的user
      request = delete("/users/1");
      mockMvc.perform(request).andExpect(content().string(equalTo("success")));

      // 7、get查一下user列表，应该为空
      request = get("/users/");
      mockMvc.perform(request)
            .andExpect(status().isOk())
            .andExpect(content().string(equalTo("[]")));


   }

}
```



## 6.ApplicationTests



```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTests {


   @Autowired
   protected WebApplicationContext wac;

   protected MockMvc mockMvc;

   @Before
   public void setup() {
      mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
   }


}
```













# 六、参考资料

1. [Spring Boot干货系列：（十二）Spring Boot使用单元测试](http://tengj.top/2017/12/28/springboot12/)
2. [Spring Boot构建RESTful API与单元测试](http://blog.didispace.com/springbootrestfulapi/)
3. [RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)

