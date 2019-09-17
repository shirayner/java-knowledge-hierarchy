[TOC]





# 前言









# 一、去除快捷方式箭头

（1）新建 `去除快捷方式箭头.bat` 文件，文件内容如下：

```bash
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 29 /d "%systemroot%\system32\imageres.dll,197" /t reg_sz /f
taskkill /f /im explorer.exe
attrib -s -r -h "%userprofile%\AppData\Local\iconcache.db"
del "%userprofile%\AppData\Local\iconcache.db" /f /q
start explorer
pause
```



（2）以管理员权限运行该bat文件即可















# 参考资料

1. [windows10 如何去掉快捷方式小箭头](https://zhidao.baidu.com/question/1242314384790353459.html)
2. 