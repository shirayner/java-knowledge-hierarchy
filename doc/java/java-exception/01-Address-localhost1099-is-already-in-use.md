

[TOC]



# 前言







# 一、Address localhost:1099 is already in use

原因是端口占用

解决方法：

cmd输入下面命令：

```bash
netstat -ano|findstr 1099
taskkill -f -pid 16784
```



![1553848397707](images/1553848397707.png)











