[TOC]







# 前言



# 二、Idea中文乱码解决

## 1.修改exe.vmoptions配置文件

> 修改idea启动编码

（1）生成用户范围的配置文件（联想maven）

依次点击 `Help` -> `Edit Custom VM Options...`，会在用户目录（例如我的是：`C:\Users\shira\.IntelliJIdea2018.3\config`）下生成 `idea64.exe.vmoptions`配置文件



（2）修改用户范围的配置文件

在生成 `idea64.exe.vmoptions`配置文件结尾追加 `-Dfile.encoding=UTF-8`



## 2.File Encodings

> 修改工程编码

（1）修改项目编码：依次点击 File -> Settings -> Editor ，修改字符编码为 UTF-8

（2）修改默认编码：依次点击 File -> Other Settings -> Settings for New Project -> Editor ，修改字符编码为 UTF-8



## 3.jvm 启动参数

设置 Tomcat 的 `VM options` 为：

```properties
-Dfile.encoding=UTF-8
```







# 参考资料

1.  [IntelliJ IDEA 乱码解决方案 （项目代码、控制台等）](https://www.cnblogs.com/vhua/p/idea_1.html)
2. [解决IDEA Tomcat乱码问题](https://www.jianshu.com/p/754aa41824b9)
3. 