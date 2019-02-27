[TOC]







# 前言

各种主流Linux发行版都采用了某种形式的包管理系统来控制软件和库的安装。





# 一、基于Red Hat 的系统

基于RedHat的系统也有几种不同的可用前端工具。常见的有以下3种：

- yum：在RedHat和Fedora中使用
- urpm：在Mandriva中使用
- zypper：在openSUSE中使用。

这些前端都是基于rpm命令行工具的。



## 1.列出已安装包

```shell
yum  list  installed
```

输出的信息可能会在屏幕上一闪而过，所以最好是将已安装包的列表重定向到一个文件中。可以用more或less命令（或一个GUI编辑器）按照需要查看这个列表。

```shell
yum list installed > installed_software
```



## 2.用yum安装软件

（1）执行如下命令，将会从仓库中安装指定的软件包、所有它需要的库以及依赖的其他包：

```shell
yum install package_name
```



（2）也可以手动下载rpm安装文件并用yum安装，这叫作本地安装。基本的命令是：

```shell
yum localinstall package_name.rpm
```





## 3.用yum更新软件

列出所有已安装包的可用更新：

````shell
yum list updates
````



更新指定软件包：

```shell
yum update package_name
```



更新所有软件包：

```shell
yum update
```





## 4.用yum卸载软件

只删除软件包而保留配置文件和数据文件：

```shell
yum remove package_name
```



删除软件和它所有的文件:

```shell
yum erase package_name
```





## 5.处理损坏的包依赖关系

有时在安装多个软件包时，某个包的软件依赖关系可能会被另一个包的安装覆盖掉。这叫作**损坏的包依赖关系**（broken dependency）。

解决方法如下：

（1）清理

```shell
yum clean all
```



（2）使用 yum 的update命令



（3）有时，只要清理了放错位置的文件就可以了。如果这还解决不了问题，试试下面的命令：

```shell
yum deplist package_name
```

这个命令显示了所有包的库依赖关系以及什么软件可以提供这些库依赖关系。一旦知道某个包需要的库，你就能安装它们了。



## 6.yum软件仓库

要想知道你现在正从哪些仓库中获取软件，输入如下命令：

```list
yum repolist
```



如果仓库中没有需要的软件，你可以编辑一下配置文件。yum的仓库定义文件位于/etc/yum.repos.d。你需要添加正确的URL，并获得必要的加密密钥。



像 `rpmfusion.org` 这种优秀的仓库站点会列出必要的使用步骤。

有时这些仓库网站会提供一个可下载的rpm文件，可以用`yum localinstall`命令进行安装。这个rpm文件在安装过程会为你完成所有的仓库设置工作。





# 二、从源码安装

安装的具体过程：

```shell
$ tar zxvf XXXX.tar.gz (or tar jxvf XXXX.tar.bz2)  # 解包
$ cd XXXX
$ ./configure     # 1.配置，建立makefile文件
$ make            # 2.编译
$ make install    # 3.安装
```



## 1.`./configure`

> 开始configure前，应该仔细阅读源码目录下的**README**或者**INSTALL**文件，此文件中列出了软件安装所需的操作。configure实际上是一个脚本文件，在当前目录中键入”./configure”,shell就会运行当前目录下的configure脚本。



生成makefile文件：

```shell
./configure

./configure  --prefix=/usr/local   # 指定程序的安装位置
```





## 2.make

> 如果configure过程正确完成，那么在源码目录，会生成相应的Makefile文件，Makefile文件简单来说包括的是一组文件依赖关系以及编译链接的相关步骤，事实上真正的编译链接工作也不是make所做的，make只是一个通用的工具，一般情况下，make会根据Makefile中的规则调用合适的编译器编译所有与当前软件相依赖的源码，生成所有相关的目标文件，最后再使用链接器生成最终的可执行程序。



编译源码：

```shell
make
```

检查是否编译成功，为0代表运行成功：

```shell
$ echo &?
0
```





## 3.`make install`

> 当上面两个步骤正确完成，代表着编译链接过程已经完全结束，最后要做的就是将可执行程序安装到正确的位置，在这个步骤，普通用户可能没有相关目录的操作权限，临时切换到root是一个不错的选择，”install”只是Makefile文件中的一个标号,”make install”代表着make工具执行Makefile文件中”install”标号下的所有相关操作，如果在configure阶段没有使用”–prefix=/opt/XXX”指定应用程序的安装目录，那么应用程序一般会被默认安装到/usr/local/bin，如果/usr/local/bin已经存在于您的PATH中，那么安装已经基本结束



开始安装：

```shell
make install
```



## 4.`make clean & make uninstall`

> 这两个步骤只是安装的后续操作，有一点必须注意，”clean”和”uninstall”也是Makefile文件中相应的两个标号，执行这两个步骤的时候Makefile文件必要保留，”make clean”用来清除编译连接过程中的一些临时文件，”make uninstall”是卸载相关应用程序，与make install类似，make uninstall也需要切换到root执行，不过”uninstall”标号在好多Makefile中都被省略掉了，朋友们完全可以自己在相应的Makefile文件一探究竟。









# 参考资料

1. [Linux 源码安装软件](https://blog.csdn.net/liupeifeng3514/article/details/79054510)
2. [Linux基础：从源码安装软件](http://blog.51cto.com/skypegnu1/1626265)
3. 





