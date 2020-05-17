[toc]







[toc]







# 前言

# 推荐阅读

> - [Spock Framework Reference Documentation](http://spockframework.org/spock/docs/1.3/index.html)
> - [Spock 单元测试实践](https://juejin.im/post/5d8ad44a51882509615bc937)
> - [Spock测试框架](http://tech.dianwoda.com/2018/11/17/spock-xlp/)
> - [使用Spock框架进行单元测试](http://blog.2baxb.me/archives/1398)
> - [Spock in Java 慢慢爱上写单元测试](https://juejin.im/post/5d982268e51d45782e6039bf)
> - [Spock与现有测试框架的对比](https://blog.csdn.net/chizeiqin5110/article/details/100861612?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)
> - 







# 一、Spock 基本概念

## 1.简介

Spock是针对Java和Groovy应用程序的测试和规范框架。得益于JUnit runner，Spock能够在大多数IDE、编译工具、持续集成服务下工作。Spock的灵感源于JUnit,jMock, RSpec, Groovy, Scala, Vulcans以及其他优秀的框架形态。



Spock 使用 given-when-then三段式语法，使代码层级分明，更具可读性

## 2.Specification 

Spock 使用 groovy 语法，所有测试类都需要继承自`Specification `，测试类结构如下：

```groovy
class MyFirstSpecification extends Specification {
  // fields 字段	
  // fixture methods  模板方法
  // feature methods  特征方法
  // helper methods 测试帮助方法
}
```



（1）定义变量

```groovy
def obj = new ClassUnderSpecification()  
def coll = new Collaborator()  
```



（2）模板方法

Spock 包含如下模板方法，也即生命周期方法

```groovy
def setupSpec() {}    // runs once -  before the first feature method  
def setup() {}        // runs before every feature method  
def cleanup() {}      // runs after every feature method  
def cleanupSpec() {}  // runs once -  after the last feature method  
```



（3）特征方法

```groovy
def "pushing an element on the stack"() {
  // blocks go here
}
```



特征方法也即测试方法，由 block 组成， 是 Specification 的核心

## 3.与Junit的对比

| Spock               | JUnit                              |
| :------------------ | :--------------------------------- |
| Specification       | Test class                         |
| `setup()`           | `@Before`                          |
| `cleanup()`         | `@After`                           |
| `setupSpec()`       | `@BeforeClass`                     |
| `cleanupSpec()`     | `@AfterClass`                      |
| Feature             | Test                               |
| Feature method      | Test method                        |
| Data-driven feature | Theory                             |
| Condition           | Assertion                          |
| Exception condition | `@Test(expected=…)`                |
| Interaction         | Mock expectation (e.g. in Mockito) |



## 4.Block

特征方法由块（block）组成，通过块实现了特征方法的每个概念阶段。

由标签来划分一个个块，如：

```
    def "Adding two and three results in 5"() {
        given: "the integers two and three"
        int a = 3
        int b = 2
        when: "they are added"
        int result = a + b
        then: "the result is five"
        result == 5
    }
```





Block 与特征方法的概念阶段映射关系如下：

![块2相](images/Blocks2Phases.png)

块将方法划分为不同的部分，共有6种块：

| Spock Block | 作用                                                    |
| ----------- | ------------------------------------------------------- |
| given       | 创建初始化条件，作用类似于setup方法                     |
| when        | 描述刺激，即表示要触发什么行为， 也即触发将被测试的操作 |
| then        | 断言，验证结果                                          |
| expect      | then的简洁用法                                          |
| where       | 参数化测试                                              |
| cleanup     | 释放资源                                                |







 示例：

```groovy
    @Unroll
    def "Query user by id=#id and username=#username"() { // 引号中为方法或块的 description
        given: "a mock user maper"
        //mock
        UserMapper userMapper = Mock(UserMapper.class)
        //stub
        userMapper.findById(_) >> User.builder().id(id+100).build();
        userMapper.findByUserName(_) >> User.builder().username(username+"100").build()

        and: "an  userService"
        def userService = new UserServiceImpl(userMapper)

        when: "query user"
        def user = userService.query(User.builder().id(id).username(username).build())

        then:
        with(user){
            user.id == resId
            user.username == resUserName
        }

        where:
        id   |  username     ||  resId  |  resUserName
        0    |  null         ||  100    |  null
        1    |  null         ||  101    |  null
        -1   |  "tom"        ||  null   |  "tom100"
        -1   |  "jack"       ||  null   |  "jack100"

    }
```





### 4.1 given

given 中主要是进行一些初始化设置，given 是setup的别名

```groovy
    @Unroll
    def "Query user by id=#id and username=#username"() {
        given: "a mock user maper"
        //mock
        UserMapper userMapper = Mock(UserMapper.class)
        //stub
        userMapper.findById(_) >> User.builder().id(id+100).build();
        userMapper.findByUserName(_) >> User.builder().username(username+"100").build()

        and: "an  userService"
        def userService = new UserServiceImpl(userMapper)

        when: "query user"
        def user = userService.query(User.builder().id(id).username(username).build())

        then:
        with(user){
            user.id == resId
            user.username == resUserName
        }

        where:
        id   |  username     ||  resId  |  resUserName
        0    |  null         ||  100    |  null
        1    |  null         ||  101    |  null
        -1   |  "tom"        ||  null   |  "tom100"
        -1   |  "jack"       ||  null   |  "jack100"

    }
```

在 given 中可以模拟对象，并进行插桩 。

注意： 当mock 一个对象后，如果执行对象的方法时，会执行一个空操作，也即并不会执行这个方法。这时我们可以为mock的这个对象进行插桩，让方法返回，我们想要的结果，如：

```java
public class UserMapper {
    void save(User u){
        system.out.println("save user");
    }
}
```



```groovy
    def "Save user"() {
        given: "mock user maper"
        //mock
        UserMapper userMapper = Mock(UserMapper.class);
        def userService = new UserServiceImpl(userMapper)
        
        when: "add user"
        userService.save(User.builder().id(1).username("tom").build())  

        then:
        1 * userMapper.save(_)   // 验证方法执行了一次
    }
```

此时并不会打印`save user `这条输出语句，但是断言通过







### 4.2 when 和 then

when 和 then 总是一起出现，描述了刺激和预期的反应。

when 中可以包含任意代码，而 then 则被限制为条件、异常条件、交互



#### 4.2.1 条件

验证条件是否满足

```groovy

given:
def stack = new Stack()
def elem = "push me"
    
when:
stack.push(elem)

then:
!stack.empty
stack.size() == 1
stack.peek() == elem
```



#### 4.2.2 异常

断言抛出了某个异常

```groovy

given:
def stack = new Stack()

when:
stack.pop()

then:
thrown(EmptyStackException)
stack.empty
```



异常条件之后可能还有其他条件（甚至其他块）， 要访问异常，需要先将其绑定到变量：

```
given:
def stack = new Stack()

when:
stack.pop()

then:
def e = thrown(EmptyStackException)
e.cause == null
```



断言异常不被抛出

```groovy
def "HashMap accepts null key"() {
  given:
  def map = new HashMap()

  when:
  map.put(null, "elem")

  then:
  notThrown(NullPointerException)
}
```



#### 4.2.3 Interactions（交互）

条件描述对象的状态，而交互描述对象之间的通信方式。（ **存疑：交互就是验证方法的调用频率** ）



#####  交互的结构

```java
class Publisher {
  List<Subscriber> subscribers = []
  int messageCount = 0
  void send(String message){
    subscribers*.receive(message)
    messageCount++
  }
}

interface Subscriber {
  void receive(String message)
}

class PublisherSpec extends Specification {
  Publisher publisher = new Publisher()
}
```





```groovy
def "events are published to all subscribers"() {
  given:
  def subscriber1 = Mock(Subscriber)
  def subscriber2 = Mock(Subscriber)
  def publisher = new Publisher()
  publisher.add(subscriber1)
  publisher.add(subscriber2)

  when:
  publisher.fire("event")

  then:
  1 * subscriber1.receive("event")
  1 * subscriber2.receive("event")
}
```



以上代码验证了，“当发布者发送“ hello”消息时，两个订阅者都应该只收到一次该消息。”，then块中包含了两个交互，每个交互都有四个不同的部分：*基数*，*目标约束*，*方法约束*和*参数约束*：

```
1 * Subscriber.receive（“hello”）
|   | 		   | 	    | 
|   | 		   | 	    参数约束
|  	| 	       方法约束
| 	目标约束
基数
```



（1）基数

交互的基数描述了期望方法调用的频率。它可以是固定数字或范围：

```groovy
1 * subscriber.receive("hello")      // exactly one call
0 * subscriber.receive("hello")      // zero calls
(1..3) * subscriber.receive("hello") // between one and three calls (inclusive)
(1.._) * subscriber.receive("hello") // at least one call
(_..3) * subscriber.receive("hello") // at most three calls
_ * subscriber.receive("hello")      // any number of calls, including zero
                                     // (rarely needed; see 'Strict Mocking')
```



（2）目标约束

交互的目标约束描述了预期哪个模拟对象接收方法调用：

```groovy
1 * subscriber.receive("hello") // a call to 'subscriber'
1 * _.receive("hello")          // a call to any mock object
```



（3）方法约束

交互的方法约束描述了期望调用哪种方法：

```java
1 * subscriber.receive("hello") // a method named 'receive'
1 * subscriber./r.*e/("hello")  // a method whose name matches the given regular expression
                                // (here: method name starts with 'r' and ends in 'e')
```

（4）参数约束

```groovy
1 * subscriber.receive("hello")        // an argument that is equal to the String "hello"
1 * subscriber.receive(!"hello")       // an argument that is unequal to the String "hello"
1 * subscriber.receive()               // the empty argument list (would never match in our example)
1 * subscriber.receive(_)              // any single argument (including null)
1 * subscriber.receive(*_)             // any argument list (including the empty argument list)
1 * subscriber.receive(!null)          // any non-null argument
1 * subscriber.receive(_ as String)    // any non-null argument that is-a String
1 * subscriber.receive(endsWith("lo")) // any non-null argument that is-a String
1 * subscriber.receive({ it.size() > 3 && it.contains('a') })
// an argument that satisfies the given predicate, meaning that
// code argument constraints need to return true of false
// depending on whether they match or not
// (here: message length is greater than 3 and contains the character a)
```





##### 交互的范围

`then:`块中声明的交互作用仅限于前一个`when:`块：

```groovy
when:
publisher.send("message1")

then:
1 * subscriber.receive("message1")

when:
publisher.send("message2")

then:
1 * subscriber.receive("message2")
```





# 三、数据驱动测试







# 四、基于交互的测试

### 3.1 模拟

#### 3.1.1 模拟静态方法

全局模拟支持静态方法的模拟和存根：

```groovy
RealSubscriber anySubscriber = GroovySpy(global: true)

1 * RealSubscriber.someStaticMethod("hello") >> 42
```

当全局模拟仅用于模拟构造函数和静态方法时，实际上不需要模拟的实例。在这种情况下，可以这样写：

```groovy
GroovySpy(RealSubscriber, global: true)
```





### 3.2  存根交互

存根是使协作者以某种方式响应方法调用的行为。

可以在通常的地方声明交互：

> - 在`then:`块内部，或`when:`块之前的任何位置
> - 如果仅将模拟对象用于存根，则通常[在模拟创建时](http://spockframework.org/spock/docs/1.3/interaction_based_testing.html#declaring-interactions-at-creation-time)或在 `given:`块中声明交互。



（1）返回固定值

使用右移（`>>`）运算符返回固定值

```groovy
subscriber.receive("message1") >> "ok"
subscriber.receive("message2") >> "fail"
```



（2）返回值序列

要在连续调用中返回不同的值，请使用三重右移（`>>>`）运算符：

```
subscriber.receive(_) >>> ["ok", "error", "error", "ok"]
```

表示第一次调用返回 "ok"，第二次和第三次调用返回 "error"，其余所有调用返回"ok"。



（3）计算返回值

要基于方法的参数计算返回值，请使用 right-shift（`>>`）运算符和闭包一起使用。如果闭包声明了多个参数或单个*类型化*参数，则方法参数将被一一映射到闭包参数

```groovy
subscriber.receive(_) >> { String message -> message.size() > 3 ? "ok" : "fail" }
```

表示如果消息长度超过三个字符，则返回"ok"，否则返回"fail"



（3）抛出异常

```
subscriber.receive(_) >> { throw new InternalError("ouch") }
```



（4）链式方法响应

```
subscriber.receive(_) >>> ["ok", "fail", "ok"] >> { throw new InternalError() } >> "ok"
```

表示在前三个调用中返回 "ok", "fail", "ok"，在第四次调用时返回 "InternalError"，并在以后的调用中返回`ok`







# 五、入门示例

## 1.引入依赖

```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.spockframework</groupId>
                <artifactId>spock-bom</artifactId>
                <version>2.0-M1-groovy-2.5</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- Mandatory dependencies for using Spock -->
        <dependency>
            <groupId>org.spockframework</groupId>
            <artifactId>spock-core</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Optional dependencies for using Spock -->
        <dependency> <!-- use a specific Groovy version rather than the one specified by spock-core -->
            <groupId>org.codehaus.groovy</groupId>
            <artifactId>groovy</artifactId>
            <version>2.5.8</version>
        </dependency>
        <dependency> <!-- enables mocking of classes (in addition to interfaces) -->
            <groupId>net.bytebuddy</groupId>
            <artifactId>byte-buddy</artifactId>
            <version>1.9.3</version>
            <scope>test</scope>
        </dependency>
        <dependency> <!-- enables mocking of classes without default constructor (together with CGLIB) -->
            <groupId>org.objenesis</groupId>
            <artifactId>objenesis</artifactId>
            <version>2.6</version>
            <scope>test</scope>
        </dependency>
        <dependency> <!-- only required if Hamcrest matchers are used -->
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>1.3</version>
            <scope>test</scope>
        </dependency>
        <!-- Dependencies used by examples in this project (not required for using Spock) -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.197</version>
        </dependency>

        <!-- Dependencies used  for generate getter and setter of entity (not required for using Spock)-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Mandatory plugins for using Spock -->
            <plugin>
                <!-- The gmavenplus plugin is used to compile Groovy code. To learn more about this plugin,
                visit https://github.com/groovy/GMavenPlus/wiki -->
                <groupId>org.codehaus.gmavenplus</groupId>
                <artifactId>gmavenplus-plugin</artifactId>
                <version>1.6</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compileTests</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <!-- Optional plugins for using Spock -->
            <!-- Only required if names of spec classes don't match default Surefire patterns (`*Test` etc.) -->
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M4</version>
                <configuration>
                    <useFile>false</useFile>
                    <includes>
                        <include>**/*Test.java</include>
                        <include>**/*Spec.java</include>
                    </includes>
                </configuration>
            </plugin>
        </plugins>
    </build>

```







