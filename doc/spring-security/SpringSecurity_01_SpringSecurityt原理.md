[TOC]





# 前言

在分析SpringSecurity之前，我们先来搭建一下调试环境，具体过程参见：

> [SpringBoot_11_Security_01_整合SpringSecurity.md](../spring-boot/SpringBoot_11_Security_01_整合SpringSecurity.md)





# 一、Spring Security 简介

## 1.什么是Spring Security

Spring Security 是一个能够为基于Spring的企业应用系统提供安全访问控制解决方案的安全框架。



Spring Security 安全框架除了包含基本的**认证**和**授权**功能外，还提供了**加解密**、**统一登录**等一系列支持。

> - **认证**（`Authentication`）：确认用户可以访问当前系统
> - **授权**（`Authorization`）：确定用户在当前系统中是否能够执行某个操作，即用户所拥有的功能权限





## 2.常用安全框架的比较





# 二、Spring Security 原理

## 1.基于Servlet过滤器（Filter）

在 Java Web 工程中， 一般**使用 Servlet 过滤器（ Filter）对请求进行拦截**，然后在Filter 中通过自己的验证逻辑来决定是否放行请求。同样地，Spring Security 也是基于这个原理，**在进入DispatcherServlet 前**就可以对Spring MVC 的请求进行拦截，然后通过一定的验证，从而决定是否放行请求访问系统。



常用过滤器







```
        WebAsyncManagerIntegrationFilter 
        SecurityContextPersistenceFilter 
        HeaderWriterFilter 
        CorsFilter 
        LogoutFilter
        RequestCacheAwareFilter
        SecurityContextHolderAwareRequestFilter
        AnonymousAuthenticationFilter
        SessionManagementFilter
        ExceptionTranslationFilter
        FilterSecurityInterceptor
        UsernamePasswordAuthenticationFilter
        BasicAuthenticationFilter
```











# 三、Spring Security 源码分析

## 1.配置类

在`spring-boot-autoconfigure`工程下 `META-INF/spring.factories` 文件中可以找到Spring Security的配置类：

```properties
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityRequestMatcherProviderAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
```

暂时不考虑reactive及oauth2，只考虑servlet，这涉及到如下配置类：

```java
SecurityAutoConfiguration
SecurityRequestMatcherProviderAutoConfiguration
UserDetailsServiceAutoConfiguration
SecurityFilterAutoConfiguration
```





















# 参考资料

1. 《深入浅出 SpringBoot 2.x》
2. [springSecurity安全框架的学习和原理解读](https://blog.csdn.net/liushangzaibeijing/article/details/81220610)
3. [Spring Security 详解](https://www.jianshu.com/p/ac42f38baf6e)
4. [spring-security权限控制详解](https://www.cnblogs.com/fp2952/p/8933107.html)







