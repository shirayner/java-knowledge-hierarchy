# Spring源码分析_04_AOP

[toc]

## 推荐阅读

> - [Spring 源码学习(八) AOP 使用和实现原理](http://www.justdojava.com/2019/07/17/spring-analysis-note-8/)
> - [SpringBoot2 | Spring AOP 原理深度源码分析（八）](https://juejin.im/post/5ce4d8e451882533591d549a)
> - [Spring 源码学习(八) AOP 使用和实现原理](https://juejin.im/post/5d2f43ab5188257e302863de)
> - [Spring源码分析-AOP](https://binglau7.github.io/2017/12/18/Spring%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-AOP/)



## 推荐项目

> - https://gitee.com/vip-augus/spring-analysis-note
> - 



## 一、Spring AOP 使用示例

### 1. 创建用于拦截的 bean

创建用于拦截的 bean

```java
public class TestBean {

	private String testStr = "testStr";

	public void testAop() {
		// 被拦截的方法，简单打印
		System.out.println("test");
	}
}
```





### 2.创建 Advisor

```java
@Aspect
public class TestBeanAspect {
    @Pointcut("execution(* *.testAop(..))")
    public void test() {

    }

    @Before("test()")
    public void beforeTest() {
        System.out.println("beforeTest");
    }

    @After("test()")
    public void afterTest() {
        System.out.println("afterTest");
    }

    @Around("test()")
    public Object aroundTest(ProceedingJoinPoint p) {
        System.out.println("brfore around");
        Object o = null;
        try {
            o = p.proceed();
        } catch (Throwable e) {
            e.printStackTrace();
        }
        System.out.println("after around");
        return o;
    }
}
```



### 3.创建配置文件

TestBeanAspect-aop.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xmlns:aop="http://www.springframework.org/schema/aop"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans
	   http://www.springframework.org/schema/beans/spring-beans.xsd
	   http://www.springframework.org/schema/aop
	   https://www.springframework.org/schema/aop/spring-aop.xsd">
	<!--开启 AOP 功能-->
	<aop:aspectj-autoproxy />

	<bean id="testBean" class="com.ray.study.spring.aop.TestBean"/>

	<bean class="com.ray.study.spring.aop.TestBeanAspect" />

</beans>
```



### 4.测试

```
@Test
public void testAop() {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("TestBeanAspect-aop.xml");
    TestBean bean = (TestBean) ctx.getBean("testBean");
    bean.testAop();
}
```



输出日志：

```
brfore around
beforeTest
test
after around
afterTest
```



















