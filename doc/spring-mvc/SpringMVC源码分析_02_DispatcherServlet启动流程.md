[TOC]





# 一、前言

上一节我们简单了解 SpringMVC 架构，这一节我们围绕前端控制器 DispatcherServlet 来详细分析一下SpringMVC启动流程。





# 二、DispatcherServlet 分析

## 1.类图



![1544771516166](images/1544771516166.png)



DispatcherServlet具有如下作用

> - 扩展自 `HttpServlet`： 因此具备servlet功能
> - 扩展自 `EnvironmentAware` ：因此能得到 Spring 通知的 Environment
> - 扩展自`EnvironmentCapable`：因此能包含和暴露  Environment 引用
> - 扩展自 `ApplicationContextAware`：因此能在启动时得到Spring 通知的  ApplicationContext





