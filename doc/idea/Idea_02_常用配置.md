# 一、前言
在上一节，我们安装并激活了IDEA，这一节我们来设置下Idea的常用配置：
> - 项目相关配置
> - Idea常用配置


# 二、项目相关配置
运行Idea，出现下图
![在这里插入图片描述](https://img-blog.csdn.net/20181008133536362?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 1.配置默认JDK
### 1.1 添加 SDKs
>（1）依次选择 Configure->Project Default ->Project Structrue ->PlatForm Settings
>
>![在这里插入图片描述](https://img-blog.csdn.net/20181008133814261?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

>（2）添加JDK
>  点击“+”，选择JDK

![在这里插入图片描述](https://img-blog.csdn.net/20181008133915356?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

> 选择JDK安装目录，点击ok，然后点击 apply ，ok，即可
>
> ![在这里插入图片描述](https://img-blog.csdn.net/20181008135222609?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

###  1.2  配置 Project SDK

![在这里插入图片描述](https://img-blog.csdn.net/20181008140200713?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2.配置默认tomcat
（1）依次选择 Configure->Project Default ->Run Configurations

![在这里插入图片描述](https://img-blog.csdn.net/20181008140330142?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


（2）依次选择 “+”->Tomcat Server->Local

（3）按下图所示配置好tomcat信息，点击apply ,ok
![在这里插入图片描述](https://img-blog.csdn.net/20181008140445267?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 3.配置默认Maven
（1）依次选择 Configure->Project Default ->Settings
（2）配置maven的安装目录，以及Setting.xml
![在这里插入图片描述](https://img-blog.csdn.net/2018100814072477?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 4.配置 Java Compiler
（1）依次选择 Configure->Project Default ->Settings
（2）给 Additional command line parameters添加-parameters参数
![在这里插入图片描述](https://img-blog.csdn.net/20181008140821394?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 5.配置字符编码
（1）依次选择 Configure->Project Default ->Settings

![在这里插入图片描述](https://img-blog.csdn.net/20181008143501363?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 6.配置新建文件文件头

通过此配置可以在每次新建类时，在文件开始处插入以下注释：

```
/**
 * @desc hello
 *
 * @author rui.shi@hand-china.com
 * @date 2018/10/8
 */
```

对应配置如下：
```
/**
 * @desc:
 * 
 * @author: rui.shi@hand-china.com
 * @date: ${DATE}
 */
```
![在这里插入图片描述](https://img-blog.csdn.net/20181008153131714?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





# 三、Idea常用配置
## 1.修改主题
可能大家会觉得软件的界面不太好看，我们可以换一下主题。选择菜单栏“File--settings--apperance--theme”，主题选择Darcula：
![在这里插入图片描述](https://img-blog.csdn.net/20181008142949841?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 2.导入第三方主题
系统提供的两种主题可能都不太好看，可访问如下网站获取第三方主题：
> - [http://www.riaway.com/](http://www.riaway.com/)
>
> ![在这里插入图片描述](https://img-blog.csdn.net/20181008142422376?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 3.修改字体
### 3.1 修改代码字体
选择菜单栏“File--settings--Editor--Font”：
![在这里插入图片描述](https://img-blog.csdn.net/20181008143122695?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 3.2 修改控制台字体

![在这里插入图片描述](https://img-blog.csdn.net/20181008143858732?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## 4.修改默认快捷键
如果想修改成Eclipse的快捷键，可以选择菜单栏"file--Settings--Keymap"：
![在这里插入图片描述](https://img-blog.csdn.net/2018100814412647?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 5.配置代码的自动提示
### 5.1 配置自动提示
新版的Idea默认具有代码自动补齐的功能（老版本的Idea是没有的），自动补齐的设置如下：
![在这里插入图片描述](https://img-blog.csdn.net/20181008144736939?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 5.2 配置代码提示的大小写区分
Idea默认的代码提示是大小写敏感的，通过如下设置可使其对大小写不敏感：
![在这里插入图片描述](https://img-blog.csdn.net/20181008145419426?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 6.设置自动导包
![在这里插入图片描述](https://img-blog.csdn.net/20181008145718706?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 7.显示代码行数
![在这里插入图片描述](https://img-blog.csdn.net/20181008145914910?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 8.禁止自动打开上次的工程
![在这里插入图片描述](https://img-blog.csdn.net/20181008150042435?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 9.禁止代码折叠
IDEA默认有很多地方的代码都会自动折叠，如果不习惯，可以取消掉。

![在这里插入图片描述](https://img-blog.csdn.net/20181008150411537?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 10.修改注释位置，禁用“语句堆一行”
![在这里插入图片描述](https://img-blog.csdn.net/20181008150711585?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 11.设置多Tab页
![在这里插入图片描述](https://img-blog.csdn.net/20181008154042348?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2OTgxMzMz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 三、参考资料
1. [Android Studio 入门级教程（一）](http://www.cnblogs.com/abao0/p/6934023.html)