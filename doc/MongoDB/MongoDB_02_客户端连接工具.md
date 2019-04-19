[TOC]





# 前言



# 一、studio3t 

studio3t是mongodb优秀的客户端工具 

官网：

> https://studio3t.com



## 1. 下载

2018.3版下载地址：

> https://download.csdn.net/download/qq_38363025/10544563



 ## 2. 安装

一路默认安装即可



## 3.激活

> - 参考：https://blog.csdn.net/justweb/article/details/83619685
> - 并非真正破解，而是通过重置studio 3t的试用时间解决的。每次开机重启脚本重置试用时间



（1）新建脚本文件`studio3t.bat`，内容如下：

```
@echo off
ECHO 重置Studio 3T的使用日期......
FOR /f "tokens=1,2,* " %%i IN ('reg query "HKEY_CURRENT_USER\Software\JavaSoft\Prefs\3t\mongochef\enterprise" ^| find /V "installation" ^| find /V "HKEY"') DO ECHO yes | reg add "HKEY_CURRENT_USER\Software\JavaSoft\Prefs\3t\mongochef\enterprise" /v %%i /t REG_SZ /d ""
ECHO 重置完成, 按任意键退出......
pause>nul
exit
```



（2）将文件`studio3t.bat`文件移动到如下路径中，实现开机自动执行上述脚本，来重置试用时间

```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp
```









# 参考资料



