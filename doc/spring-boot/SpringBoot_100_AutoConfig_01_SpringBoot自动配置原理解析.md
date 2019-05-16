SpringBoot内置的自动化配置选项可见：

> - [SpringBoot Refrerence_Appendix A. Common application properties](https://docs.spring.io/spring-boot/docs/2.1.5.RELEASE/reference/htmlsingle/#common-application-properties)
> - 





- 项目启动 
- @EnableAutoConfiguration 的作用，会从项目依赖的所有jar包中寻找META-INF/spring.factories中，找到自动配置类，然后根据配置类实例化相应的bean，再加入到容器中
- 

