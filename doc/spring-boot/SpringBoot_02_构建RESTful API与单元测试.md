[TOC]



# 前言

上一节 `SpringBoot`快速入门中 ，我们实现了一个简单的请求响应流程。



下面我们将尝试使用Spring MVC来实现一组对User对象操作的RESTful API，配合注释详细说明在Spring MVC中如何映射HTTP请求、如何传参、如何编写单元测试。





# 一、构建`Restful API`及其测试

## 1.创建子模块

### 1.1 创建项目

这里我们创建一个项目，创建步骤同上一节。

```properties
group = 'com.ray.study'
artifact ='spring-boot-02-restful-test'
```



### 1.2 引入依赖

在父工程`spring-boot-seeds` 的 `settings.gradle`加入子工程

```properties
rootProject.name = 'spring-boot-seeds'
include 'spring-boot-01-helloworld'
include 'spring-boot-02-restful-test'
```



这样，子工程`spring-boot-02-restful-test`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
        implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```





## 2.`RESTful API`

### 2.1 `RESTful API`设计

| 方法名 | 请求类型 | URL         | 功能说明       |
| ------ | -------- | ----------- | -------------- |
| list   | GET      | /users/     | 查询用户列表   |
| get    | GET      | /users/{id} | 根据id查询用户 |
| insert | POST     | /users/     | 新增用户       |
| update | PUT      | /users/     | 更新用户       |
| delete | DELETE   | /users/{id} | 根据id删除用户 |



### 2.2  `RESTful API`实现

#### 2.2.1 User

实体类如下

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

	private Long id;

	private String name;

	private Integer age;
}
```



#### 2.2.2 `UserService`

（1）`UserService`

```java
public interface UserService {

	/**
	 * 获取用户列表
	 * @return
	 */
	List<User> list();

	/**
	 * 根据id获取用户
	 * @param id
	 * @return
	 */
	User get(Long id);

	/**
	 * 新增用户
	 * @param user
	 * @return
	 */
	User insert(User user);

	/**
	 * 更新用户
	 * @param user
	 * @return
	 */
	User update(User user);

	/**
	 * 删除用户
	 * @param id
	 * @return
	 */
	boolean delete(Long id);

}
```



（2）`UserServiceImpl`

```java
@Service
public class UserServiceImpl  implements UserService {

	/** 创建线程安全的Map **/
	private static Map<Long, User> users = Collections.synchronizedMap(new HashMap<>());

	@Override
	public List<User> list() {
		return new ArrayList<>(users.values());
	}

	@Override
	public User get(Long id) {
		return users.get(id);
	}

	@Override
	public User insert(User user) {
		users.put(user.getId(), user);
		return user;
	}



	@Override
	public User update(User user) {
		User u = users.get(user.getId());
		u.setName(user.getName());
		u.setAge(user.getAge());
		users.put(user.getId(), u);
		return u;
	}


	@Override
	public boolean delete(Long id) {
		users.remove(id);
		return true;
	}

}

```





#### 2.2.2 `UserController`



```java
@RestController
@RequestMapping(value = "/users")     // 通过这里配置使下面的映射都在/users下
public class UserController {
	@Autowired
	UserService userService;


	/**
	 * 获取用户列表：
	 * 		处理"/users/"的GET请求，用来获取用户列表
	 * 		还可以通过@RequestParam从页面中传递参数来进行查询条件或者翻页信息的传递
	 *
	 * @return
	 */
	@GetMapping("/")
	public List<User> list() {
		return userService.list();
	}


	/**
	 * 获取用户信息：
	 * 		处理"/users/{id}"的GET请求，用来获取url中id值的User信息
	 * 		url中的id可通过@PathVariable绑定到函数的参数中
	 * @param id
	 * @return
	 */
	@GetMapping("/{id}")
	public User get(@PathVariable Long id) {
		return userService.get(id);
	}


	/**
	 * 创建用户：
	 *  	处理"/users/"的POST请求，用来创建User
	 * 		除了@ModelAttribute绑定参数之外，还可以通过@RequestParam从页面中传递参数
	 * @param user
	 * @return
	 */
	@PostMapping("/")
	public User insert(@RequestBody User user) {
		return userService.insert(user);
	}


	/** 更新用户
	 * 		处理"/users"的PUT请求，用来更新User信息
	 * @param user
	 * @return
	 */
	@PutMapping("/")
	public User update(@RequestBody User user) {
		return userService.update(user);
	}

	/**
	 * 删除用户
	 * 		处理"/users/{id}"的DELETE请求，用来删除User
	 * @param id
	 * @return
	 */
	@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
	public String delete(@PathVariable Long id) {
		boolean result = userService.delete(id);
		return "success";
	}

}
```







## 3.单元测试

关于单元测试，一般做service层测试验证逻辑的正确性即可，如若需要，也可做controller层测试。

### 3.1 `service`层测试

`service`层测试主要使用`Junit`

```java
import com.ray.study.springboot02.restfultest.model.User;
import com.ray.study.springboot02.restfultest.service.UserService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.hamcrest.CoreMatchers.notNullValue;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.is;
import static org.hamcrest.core.IsEqual.equalTo;
import static org.hamcrest.core.IsNull.nullValue;


/**
 * description
 *
 * @author shira 2019/04/28 17:26
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceImplTest {

	@Autowired
	UserService userService;

	@Test
	public void addUser() {
		User user = new User();
		user.setId(1L);
		user.setName("张三");
		user.setAge(23);

		user = userService.insert(user);
		assertThat(user.getId(),is(notNullValue()));

	}

	@Test
	public void testAll(){
		// 1. add
		User user = new User();
		user.setId(1L);
		user.setName("张三");
		user.setAge(23);

		user = userService.update(user);
		assertThat(user.getId(),is(notNullValue()));
		assertThat(user.getName(), equalTo("张三"));


		// 2.list
		List<User>  userList= userService.list();
		assertThat(userList.size(),is(1));
		//assertThat(userList,is(not(empty())));


		// 3.get
		User user1 = userService.get(user.getId());
		assertThat(user1.getName(),equalTo("张三"));


		// 4.update
		user.setName("李四");
		userService.update(user);
		User user2 = userService.get(user.getId());
		assertThat(user2.getName(), equalTo("李四"));

		// 5.delete
		userService.delete(user.getId());
		User user3 = userService.get(user.getId());
		assertThat(user3, is(nullValue()));

	}
}
```





### 3.2 `controller`层测试

`controler`层的测试主要使用`MockMvc`

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.ray.study.springboot02.restfultest.model.User;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.RequestBuilder;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import static org.hamcrest.core.IsEqual.equalTo;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

/**
 * description
 *
 * @author shira 2019/05/05 19:01
 */

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserControllerTest {

	@Autowired
	private WebApplicationContext wac;

	private MockMvc mvc;

	private ObjectMapper objectMapper = new ObjectMapper();

	@Before
	public void setUp() throws Exception {
		//mvc = MockMvcBuilders.standaloneSetup(new UserController()).build();
		mvc = MockMvcBuilders.webAppContextSetup(wac).build(); //初始化MockMvc对象
	}


	@Test
	public void getUserList() throws Exception {

		RequestBuilder request = get("/users/");
		mvc.perform(request)
				.andExpect(status().isOk());

	}

	@Test
	public void postUser() throws Exception {
		User user =  new User(1L, "测试大师", 20);
		String postJson = objectMapper.writeValueAsString(user);

		RequestBuilder request = post("/users/")
				.contentType(MediaType.APPLICATION_JSON_UTF8)
				.content(postJson);

		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.name").value("测试大师"))
				.andDo(print());
	}


	@Test
	public void testAll() throws Exception {
		RequestBuilder request;

		// 1、list查一下user列表，应该为空
		request = get("/users/");
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("[]")));

		// 2、post提交一个user
		User user =  new User(1L, "测试大师", 20);
		String postJson = objectMapper.writeValueAsString(user);
		request = post("/users/")
				.contentType(MediaType.APPLICATION_JSON_UTF8)
				.content(postJson);
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.name").value("测试大师"))
				.andDo(print());

		// 3、list获取user列表，应该有刚才插入的数据
		request = get("/users/");
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.length()").value(1));


		// 4、put修改id为1的user
		User user1 =  new User(1L, "测试大师1", 21);
		String postJson1 = objectMapper.writeValueAsString(user1);
		request = put("/users/")
				.contentType(MediaType.APPLICATION_JSON_UTF8)
				.content(postJson1);
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.name").value("测试大师1"))
				.andDo(print());

		// 5、get一个id为1的user
		request = get("/users/1");
		mvc.perform(request)
				.andExpect(content().string(equalTo(postJson1)));

		// 6、del删除id为1的user
		request = delete("/users/1");
		mvc.perform(request)
				.andExpect(status().isOk());

		// 7、get查一下user列表，应该为空
		request = get("/users/");
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("[]")));
	}
	
}
```





# 

# 二、单元测试总结

## 1.`Hamcrest `

JUnit 4.4 结合 Hamcrest 提供了一个全新的断言语法——assertThat。程序员可以只使用 assertThat 一个断言语句，结合 Hamcrest 提供的匹配符，就可以表达全部的测试思想.

关于` Hamcrest`可参见：

> - [Hamcrest Tutorial](http://hamcrest.org/JavaHamcrest/tutorial)
> - [Java Hamcrest API Reference Documentation](http://hamcrest.org/JavaHamcrest/javadoc/)



### 1.1 常见匹配器

#### 1.1.1 Core

> `anything` - always matches, useful if you don’t care what the object under test is
>
> `describedAs` - decorator to adding custom failure description
>
> `is` - decorator to improve readability - see “Sugar”, below



#### 1.1.2 Logical

> `allOf` - matches if all matchers match, short circuits (like Java &&)
>
> `anyOf` - matches if any matchers match, short circuits (like Java || )
>
> `not` - matches if the wrapped matcher doesn’t match and vice versa



#### 1.1.3 Object

> `equalTo` - test object equality using Object.equals
>
> `hasToString` - test Object.toString
>
> `instanceOf`, `isCompatibleType` - test type
>
> `notNullValue`, `nullValue` - test for null
>
> `sameInstance` - test object identity



#### 1.1.4 Beans

> `hasProperty` - test JavaBeans properties



#### 1.1.5 Collections

> `array` - test an array’s elements against an array of matchers
>
> `hasEntry`, `hasKey`, `hasValue` - test a map contains an entry, key or value
>
> `hasItem`, `hasItems` - test a collection contains elements
>
> `hasItemInArray` - test an array contains an element



#### 1.1.6 Number

> `closeTo` - test floating point values are close to a given value
>
> `greaterThan`, `greaterThanOrEqualTo`, `lessThan`, `lessThanOrEqualTo` - test ordering



#### 1.1.7 Text

> `equalToIgnoringCase` - test string equality ignoring case
>
> `equalToIgnoringWhiteSpace` - test string equality ignoring differences in runs of whitespace
>
> `containsString`, `endsWith`, `startsWith` - test string matching



### 1.2 常见示例 



```java
package com.ray.study.springboot02.restfultest.service.impl;

import com.ray.study.springboot02.restfultest.model.User;
import org.junit.Test;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;
import static org.hamcrest.collection.IsMapContaining.hasEntry;
import static org.hamcrest.collection.IsMapContaining.hasKey;
import static org.hamcrest.core.IsCollectionContaining.hasItem;
import static org.hamcrest.core.IsEqual.equalTo;
import static org.hamcrest.core.IsNull.notNullValue;
import static org.hamcrest.core.IsNull.nullValue;
import static org.hamcrest.core.StringContains.containsString;
import static org.hamcrest.number.IsCloseTo.closeTo;
import static org.hamcrest.text.IsEqualIgnoringCase.equalToIgnoringCase;

/**
 * description
 *
 * @author shira 2019/05/06 13:40
 */

public class HamcrestTest {

	/**
	 * 1.测试核心匹配符的用法： is / anything
	 */
	@Test
	public void testCore(){
		String name ="tomcat";
		int n = 100;
		double d = 100.12;

		// is 是 is(equalTo(value)) 的缩写
		assertThat(name, is("tomcat"));
		assertThat(n, is(100));
		assertThat(d, is(100.12));

		// anything: 无论什么条件，测试都通过: Creates a matcher that always matches
		assertThat(d, anything(name));
		assertThat(d, anything());

	}


	/**
	 * 2.测试逻辑匹配符的用法：allOf / anyOf / not
	 */
	@Test
	public void testLogical(){
		double d = 3.35;

		// 与
		assertThat(d, allOf(greaterThan(3.0), lessThan(3.5)));   //  区间：(3.0,3.5)
		// 或
		assertThat(d, anyOf(greaterThan(3.3), lessThan(3.2)));   //  区间：d<3.2, d>3.3
		// 非，是 not(equalTo(x)) 的缩写
		assertThat(d, is(not(3.3)));   //  区间：d<3.2, d>3.3

	}



	/**
	 * 3.测试对象匹配符的相关用法
	 */
	@Test
	public void testObject(){

		User user = new User();
		user.setId(1L);

		assertThat(user.getId(),is(notNullValue()));
		assertThat(user.getName(), is(nullValue()));

	}


	/**
	 * 4.测试文本匹配的相关用法
	 */
	@Test
	public void testText(){
		String name = "tomcat";

		// 文本相等
		assertThat(name, equalTo("tomcat"));
		assertThat(name, equalToIgnoringCase("TomCat"));

		// 文本包含
		assertThat(name, containsString("ca"));
		assertThat(name, startsWith("tom"));
		assertThat(name, endsWith("cat"));
	}


	/**
	 * 5.测试数值匹配的相关用法
	 */
	@Test
	public void testNumber(){

		double d = 3.35;

		assertThat(d, closeTo(3.0, 0.5));   //closeTo：浮点型变量的值在3.0±0.5范围内，测试通过
		assertThat(d, greaterThan(3.0));
		assertThat(d, lessThan(3.5));
		assertThat(d, greaterThanOrEqualTo(3.3));
		assertThat(d, lessThanOrEqualTo(3.4));
	}


	/**
	 * 6.测试集合匹配的相关用法
	 */
	@Test
	public void testCollection(){
		User user1 = new User(1L, "tomcat", 21);
		User user2 = new User(2L, "springboot", 22);
		User user3 = new User(2L, "springboot", 22);

		List<User> userList = new ArrayList<>();
		userList.add(user1);
		userList.add(user2);

		Map<String, User> userMap = new HashMap<>();
		userMap.put(user1.getName(), user1);
		userMap.put(user2.getName(), user2);

		// list
		assertThat(userList, hasItem(user1));
		assertThat(userList, hasItem(user3));
		assertThat(userList.size(), is(2));
		assertThat(userList,is(not(empty())));

		// map
		assertThat(userMap, hasEntry(user1.getName(),user1));
		assertThat(userMap, hasKey(user1.getName()));
		assertThat(userMap, hasValue(user1));
	}

}



```



## 2. `MockMvc`

### 2.1 入门示例

#### 2.1.1 get

主要内容：

> - 初始化`MockMVC`
> - 构造get请求
> - 执行请求
> - 响应的期望

```java
package com.ray.study.springboot02.restfultest.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.RequestBuilder;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

/**
 * description
 *
 * @author shira 2019/05/06 14:48
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class MockMvcTest {

	@Autowired
	private WebApplicationContext wac;

	private MockMvc mvc;

	private ObjectMapper objectMapper = new ObjectMapper();
	
	@Before
	public void setUp() throws Exception {
		//mvc = MockMvcBuilders.standaloneSetup(new UserController()).build();
		mvc = MockMvcBuilders.webAppContextSetup(wac).build(); //初始化MockMvc对象
	}

	@Test
	public void getUserList() throws Exception {

        // 构造get请求
		RequestBuilder request = get("/users/");
        
        // 执行get请求
		mvc.perform(request)
				.andExpect(status().isOk());  // 对请求结果进行期望，响应的状态为200

	}

}

```



#### 2.1.2 post

示例1：

```java
	@Test
	public void postUser() throws Exception {
		User user =  new User(1L, "测试大师", 20);
		String postJson = objectMapper.writeValueAsString(user);

		RequestBuilder request = post("/users/")
				.contentType(MediaType.APPLICATION_JSON_UTF8)
				.content(postJson);      // 提交Json请求参数

		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.name").value("测试大师"))  // jsonPath取值
				.andDo(print());  // 打印响应结果
	}

```



示例2：

```java
	@Test
	public void postUser2() throws Exception {
		User user =  new User(1L, "测试大师", 20);
		String postJson = objectMapper.writeValueAsString(user);

		RequestBuilder request = post("/users/")
				.contentType(MediaType.APPLICATION_JSON_UTF8)
				.content(postJson);      // 提交Json请求参数

		MvcResult mvcResult = mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.name").value("测试大师"))  // jsonPath取值
				.andDo(print())  // 打印响应结果
				.andReturn();    // 返回响应结果


		MockHttpServletResponse response = mvcResult.getResponse();
		//拿到响应状态码
		int status = response.getStatus();
		//拿到结果
		String contentAsString = response.getContentAsString();
	}

```



#### 2.1.3 put 

```java
	@Test
	public void putUser() throws Exception {
		// 4、put修改id为1的user
		User user1 =  new User(1L, "测试大师1", 21);
		String postJson1 = objectMapper.writeValueAsString(user1);
		RequestBuilder request = put("/users/")
				.contentType(MediaType.APPLICATION_JSON_UTF8)
				.content(postJson1);
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.name").value("测试大师1"))
				.andDo(print());


		// 5、get一个id为1的user
		request = get("/users/1");
		mvc.perform(request)
				.andExpect(content().string(equalTo(postJson1)));
	}

```



#### 2.1.4 delete

```java
	@Test
	public void deleteUser() throws Exception {
		RequestBuilder request = delete("/users/1");
		mvc.perform(request)
				.andExpect(status().isOk());

		// 7、get查一下user列表，应该为空
		request = get("/users/");
		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("[]")));
	}
```









### 2.2 `MockMvc`抽象

使用 `MockMvc` 可对 `controller`层进行测试

>  MockMvc实现了对Http请求的模拟，能够直接使用网络的形式，转换到Controller的调用，这样可以使得测试速度快、不依赖网络环境，而且提供了一套验证的工具，这样可以使得请求的验证统一而且很方便。



既然要模拟Http请求，那么必然就会涉及到如下 `http `元素：

> - 构造请求：对应工具为 `MockMvcRequestBuilders`、`MockHttpServletRequestBuilder`
>
>     - 请求方式：通过`MockMvcRequestBuilders`的 `get`/`post`/`put`/`delete`方法可模拟对应的http请求方式
>     - 请求头：通过`MockMvcRequestBuilders##head`方法，可设置请求头
>     - 请求体：通过`MockHttpServletRequestBuilder##content`方法，可设置请求体
>     - multipart : 通过`MockMvcRequestBuilders##multipart`方法，可设置文件上传
>
>     
>
> - 发送请求：工具为`MockMvc`
>
>     - 发送请求：通过 `MockMvc##perform`方法，可模拟发送http请求
>
>     
>
> - 响应：工具为：`ResultActions`、`MvcResult`
>
>     - 响应的期望：通过`ResultActions##andExpect`方法可对响应结果设置期望，具体的期望通过`MockMvcResultMatchers`进行匹配
>     - 执行通用操作：通过`ResultActions##andDo`方法可对响应结果设进行一些通用操作
>     - 获取响应结果：`ResultActions##andReturn`方法会返回一个`MvcResult`，其包含了响应的所有内容





### 2.3 处理响应

#### 2.3.1 `jsonPath`取值

具体用法可官方文档，很详细

> - [json-path/JsonPath](https://github.com/json-path/JsonPath)



操作符：

| Operator                  | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| `$`                       | 根节点：The root element to query. This starts all path expressions. |
| `@`                       | 当前节点：The current node being processed by a filter predicate. |
| `*`                       | 通配符：Wildcard. Available anywhere a name or numeric are required. |
| `..`                      | Deep scan. Available anywhere a name is required.            |
| `.<name>`                 | 子节点：Dot-notated child                                    |
| `['<name>' (, '<name>')]` | 一个或多个子节点：Bracket-notated child or children          |
| `[<number> (, <number>)]` | 一个或多个数组下标：Array index or indexes                   |
| `[start:end]`             | 数组片段：Array slice operator                               |
| `[?(<expression>)]`       | 过滤器表达式：Filter expression. Expression must evaluate to a boolean value. |



示例：

```json
{
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            },
            {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99
            },
            {
                "category": "fiction",
                "author": "Herman Melville",
                "title": "Moby Dick",
                "isbn": "0-553-21311-3",
                "price": 8.99
            },
            {
                "category": "fiction",
                "author": "J. R. R. Tolkien",
                "title": "The Lord of the Rings",
                "isbn": "0-395-19395-8",
                "price": 22.99
            }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    },
    "expensive": 10
}
```





| JsonPath (click link to try)            | Result                                                       |
| --------------------------------------- | ------------------------------------------------------------ |
| `$.store.book[*].author`                | The authors of all books                                     |
| `$..author`                             | All authors                                                  |
| `$.store.*`                             | All things, both books and bicycles                          |
| `$.store..price`                        | The price of everything                                      |
| `$..book[2]`                            | The third book                                               |
| `$..book[-2]`                           | The second to last book                                      |
| `$..book[0,1]`                          | The first two books                                          |
| `$..book[:2]`                           | All books from index 0 (inclusive) until index 2 (exclusive) |
| `$..book[1:2]`                          | All books from index 1 (inclusive) until index 2 (exclusive) |
| `$..book[-2:]`                          | Last two books                                               |
| `$..book[2:]`                           | Book number two from tail                                    |
| `$..book[?(@.isbn)]`                    | All books with an ISBN number                                |
| `$.store.book[?(@.price < 10)]`         | All books in store cheaper than 10                           |
| `$..book[?(@.price <= $['expensive'])]` | All books in store that are not "expensive"                  |
| `$..book[?(@.author =~ /.*REES/i)]`     | All books matching regex (ignore case)                       |
| `$..*`                                  | Give me every thing                                          |
| `$..book.length()`                      | The number of books                                          |



#### 2.3.2 打印响应

```java
	@Test
	public void postUser() throws Exception {
		User user =  new User(1L, "测试大师", 20);
		String postJson = objectMapper.writeValueAsString(user);

		RequestBuilder request = post("/users/")
				.contentType(MediaType.APPLICATION_JSON_UTF8)
				.content(postJson);      // 提交Json请求参数

		mvc.perform(request)
				.andExpect(status().isOk())
				.andExpect(jsonPath("$.name").value("测试大师"))  // jsonPath取值
				.andDo(print());  // 打印响应结果
	}
```











# 参考资料

## restful

1. [Spring Boot干货系列：（十二）Spring Boot使用单元测试](http://tengj.top/2017/12/28/springboot12/)
2. [Spring Boot构建RESTful API与单元测试](http://blog.didispace.com/springbootrestfulapi/)
3. [RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)



## 单元测试

1. [Junit 5官方文档中文版](https://doczhcn.gitbook.io/junit5/)
2. [assertThat用法](https://langgufu.iteye.com/blog/1893927)
3. [Hamcrest Tutorial](http://hamcrest.org/JavaHamcrest/tutorial)
4. [Hamcrest API](http://hamcrest.org/JavaHamcrest/javadoc/2.1/)
5. [学习 Spring Boot：（二十九）Spring Boot Junit 单元测试](https://blog.wuwii.com/springboot-test.html)
6. 

