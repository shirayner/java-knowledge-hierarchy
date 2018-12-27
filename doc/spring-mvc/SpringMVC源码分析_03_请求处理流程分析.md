[TOC]





# 一、前言

前面我们简单了解 SpringMVC 架构，这一节我们围绕前端控制器 `DispatcherServlet` 来详细分析一下SpringMVC的请求处理流程。





# 二、DispatcherServlet 分析

## 1.类图



![1544771516166](images/1544771516166.png)









# 三、流程图



## 1.请求处理流程图



![img](images/SpringMVC%E6%B5%81%E7%A8%8B%E5%9B%BE.png?lastModify=1545015684)



SpringMVC 核心组件

| 组件 Bean 类型                        | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| HandlerMapping                        | 映射请求（Request）到处理器（Handler）加上其关联的拦截器 （HandlerInterceptor）列表，其映射关系基于不同的 HandlerMapping 实现的一些 标准细节。其中两种主要 HandlerMapping 实现， RequestMappingHandlerMapping 支持标注 @RequestMapping 的方法， SimpleUrlHandlerMapping 维护精确的URI 路径与处理器的映射 |
| HandlerAdapter                        | 帮助 DispatcherServlet 调用请求处理器（Handler），无需关注其中实际的调用 细节。比如，调用注解实现的 Controller 需要解析其关联的注解. HandlerAdapter 的主要目的是为了屏蔽与 DispatcherServlet 之间的诸多细节。 |
| HandlerExceptionResolver              | 解析异常，可能策略是将异常处理映射到其他处理器（Handlers） 、或到某个 HTML 错误页面，或者其他。 |
| ViewResolver                          | 从处理器（Handler）返回字符类型的逻辑视图名称解析出实际的 View 对象，该对 象将渲染后的内容输出到HTTP 响应中。 |
| LocaleResolver, LocaleContextResolver | 从客户端解析出 Locale ，为其实现国际化视图。                 |
| MultipartResolver                     | 解析多部分请求（如 Web 浏览器文件上传）的抽象实现            |



## 2.请求处理时序图













# 四、请求处理流程分析











