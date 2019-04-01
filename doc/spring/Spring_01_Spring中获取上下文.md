[TOC]



# 前言



# 一、Spring中获取ApplicationContext

## 1. Spring范围内

在Spring范围内，可使用`@Autowired`直接注入

```java
    @Autowired
    ApplicationContext applicationContext;
```



## 2.Spring范围外

### 2.1 ApplicationContextAware

```java
@Component
public class ApplicationContextHolder  implements ApplicationContextAware {

	private  static ApplicationContext applicationContext;
	
	@Override
	public void setApplicationContext(ApplicationContext context)  {
		ApplicationContextHolder.applicationContext = context;
	}

	public static ApplicationContext getApplicationContext(){
		return applicationContext;
	}
}

```





### 2.2 通过Request获取



```java
/** web.xml中 DispatcherServlet 的 servlet-name  **/
private static final String SPRING_MVC_SERVLET_NAME = "appServlet";  

/**  org.springframework.web.servlet.FrameworkServlet.CONTEXT.appServlet  **/
private static final String APPLICATION_CONTEXT_KEY = FrameworkServlet.SERVLET_CONTEXT_PREFIX +SPRING_MVC_SERVLET_NAME;


ApplicationContext applicationContext = (ApplicationContext) httpServletRequest.getSession().getServletContext().getAttribute(APPLICATION_CONTEXT_KEY);

```





### 2.3 RequestContextUtils

```java
ApplicationContext applicationContext = (ApplicationContext) RequestContextUtils.findWebApplicationContext(httpServletRequest);
```



### 2.4 ContextLoader

```java
ApplicationContext applicationContext = (ApplicationContext) ContextLoader.getCurrentWebApplicationContext();
```



# 二、Spring 中获取Request/Response

## 1.RequestContextHolder

```
HttpServletRequest httpServletRequest = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();

		HttpServletResponse httpServletResponse = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getResponse();

```











# 参考资料

1. [spring里头各种获取ApplicationContext的方法](https://blog.csdn.net/xieyuooo/article/details/8473503)