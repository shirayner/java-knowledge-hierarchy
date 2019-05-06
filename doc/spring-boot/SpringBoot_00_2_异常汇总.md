[TOC]

# 一、异常汇总

## 1.idea中lombok的get/set报红

### 1.1 异常现象





### 1.2 异常原因

可能有如下三种原因：

> - 未添加lombok插件
> - 未开启Enable annotation processing
> - pom.xml中加入的lombok依赖包版本和自动安装的plugin中的lombok依赖包版本不一致



### 1.3 异常解决

第1、2种情况都好说。

我遇到的是第三种情况，于是我卸载`lombok plugin`，重新搜索插件`lombok` ，发现`lombok plugin`改名为`lombok`了，应该是此插件有了较大更新，于是再安装一下`lombok`，问题就解决了。

