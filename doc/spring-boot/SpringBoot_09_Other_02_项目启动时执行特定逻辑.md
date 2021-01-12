# 项目启动时执行特定逻辑

[toc]



## 推荐阅读

> - [SpringBoot - 启动时实现预加载自动执行代码](https://blog.csdn.net/qq_26878363/article/details/104500263)





## 一、项目启动时执行特定逻辑



### 1. Java自身的启动时加载方式

> - static代码块： static静态代码块，在类加载的时候即自动执行
> - 构造方法： 在对象初始化时执行。执行顺序在static静态代码块之后。



### 2.Spring启动时加载方式

> - @PostConstruct： PostConstruct注解使用在方法上，这个方法在对象依赖注入初始化之后执行。
> - ApplicationRunner和CommandLineRunner： 在Spring容器启动完成后执行，这两个接口功能基本一致，其区别在于run方法的入参



### 3.总结

Spring应用启动过程中，肯定是要自动扫描有@Component注解的类，加载类并初始化对象进行自动注入。

加载类时首先要执行static静态代码块中的代码，之后再初始化对象时会执行构造方法。

在对象注入完成后，调用带有@PostConstruct注解的方法。

当容器启动成功后，再根据@Order注解的顺序调用CommandLineRunner和ApplicationRunner接口类中的run方法。

因此，加载顺序为static>constructer>@PostConstruct>CommandLineRunner和ApplicationRunner.



