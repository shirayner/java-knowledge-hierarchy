[TOC]





# 前言







# 一、环境变量

## 1.什么是环境变量

> `bash shell` 用一个叫作环境变量（`environment variable`）的特性来存储有关 shell 会话和工作环境的信息



在 `bash shell` 中，环境变量分为两类：

- 全局变量： 对于shell会话和所有生成的子shell都是可见的
- 局部变量： 只对创建它们的shell可见



### 1.1 查看全局变量

Linux系统在你开始bash会话时就设置了一些全局环境变量。

系统环境变量基本上都是使用**全大写字母**，以区别于普通用户的环境变量。

```bash
env  			 # 列出全部环境变量
printenv HMOE	 # 列出全部环境变量 / 输出指定环境变量的值
echo $HOME       # 输出指定环境变量的值。注：$变量名可作为命令行参数
```



### 1.2 查看局部环境变量

局部环境变量只能在定义它们的进程中可见



set命令会显示为某个特定进程设置的所有环境变量，包括局部变量、全局变量以及用户定义变量。

```
set 
```



## 2.设置用户定义变量

### 2.1 设置局部用户定义变量

> - 一旦启动了bashshell（或者执行一个shell脚本），就能创建在这个shell进程内可见的局部变量了。
> - 可以通过等号给环境变量赋值，值可以是数值或字符串。

```shell
$ my_variable=Hello
$ echo  $my_variable
```



> - 系统环境变量要用大写字母，用户自定义的局部环境变量需用小写字母
> - 变量名、等号和值之间没有空格



局部变量的生命周期：

> 局部环境变量仅在定义它们的当前进程中可用，在其子shell中不可用，在其父进程中也不可用



### 2.2 设置全局环境变量

生命周期：

> - 在设定全局环境变量的进程所创建的子进程中，该变量都是可见的



创建全局环境变量的方法是先创建一个局部环境变量，然后再把它导出到全局环境中。

```shell
$ my_variable =" I am Global now" 
$ 
$ export my_variable    # 导出到全局环境中
$ echo $ my_variable 
I am Global now 
$ 
$ bash    # 使用bash命令启动一个子shell
$ 
$ echo  my_variable 
I am Global now
```



> 子shell可以重新定义变量，但无法使用export命令改变父shell中全局环境变量的值



## 3 删除环境变量

```shell
$ echo $my_variable 
I am Global now 
$ 
$ unset my_variable      # 删除全局环境变量
$ 
$ echo $my_variable 

$
```



> - 如果要用到变量，使用$；如果要操作变量，不使用$
>
> - 在子进程中删除全局环境变量，这只对子进程有效。该全局环境变量在父进程中依然可用。





## 4. 设置PATH环境变量

> 当你在shell命令行界面中输入一个外部命令时，shell必须搜索系统来找到对应的程序。

PATH环境变量定义了用于进行命令和程序查找的目录。

```shell
$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/ray/.local/bin:/home/ray/bin
$
$ PATH=$PATH:/home/ray/Scripts   # 通过引用PATH变量的方式来追加PATH环境变量
```

> PATH中的目录使用冒号分隔



通过命令行对PATH变量的修改，只能持续到退出或重启系统



## 5.环境变量持久化

### 5.1 启动文件

启动文件或环境文件 :

> 在你登入Linux系统启动一个`bashshell`时，默认情况下bash会在几个文件中查找命令。这些文件叫作启动文件或环境文件。



bash检查的启动文件取决于你启动bashshell的方式。启动bashshell有3种方式：

> - 登录时作为默认登录shell
>
> - 作为非登录shell的交互式shell
> - 作为运行脚本的非交互shell



#### 5.1.1 登录shell

> 当你登录Linux系统时，bashshell会作为登录shell启动。



登录shell会从5个不同的启动文件里读取命令：

- `/etc/profile`
- `$HOME/.bash_profile`
- `$HOME/.bashrc`
- `$HOME/.bash_login`
- `$HOME/.profile`



（1）`/etc/profile` 文 件

`/etc/profile`文件是 `bashshell` 默认的的主启动文件。只要你登录了Linux系统，bash 就会执行 `/etc/profile` 启动文件中的命令。

> 每个发行版的/etc/profile文件都有不同的设置和命令。



（2）$HOME目录下的启动文件

提供一个用户专属的启动文件来定义该用户所用到的环境变量。

> 大多数Linux发行版只用这四个启动文件中的一到两个：
>
> - `$HOME/.bash_profile`
> - `$HOME/.bashrc`
> - `$HOME/.bash_login`
> - `$HOME/.profile`
>
> 这四个文件都以点号开头，这说明它们是隐藏文件。
>
>



shell会按照按照下列顺序，运行第一个被找到的文件，余下的则被忽略：

> - $HOME/.bash_profile
>
> - $HOME/.bash_login
>
> - $HOME/.profile
>
> 这个列表中并没有$HOME/.bashrc文件。这是因为该文件通常通过其他文件运行的。



```shell
[ray@localhost ~]$ cat $HOME/.bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
[ray@localhost ~]$ 

```

`.bash_profile`启动文件会先去检查HOME目录中是不是还有一个叫`.bashrc`的启动文件。如果有的话，会先执行启动文件里面的命令。



#### 5.1.2 交互式shell进程

交互式shell：

> 如果你的`bashshell`不是登录系统时启动的（比如是在命令行提示符下敲入bash时启动），那么你启动的shell叫作交互式shell。



如果bash是作为交互式shell启动的，它就不会访问 `/etc/profile` 文件，只会检查用户HOME目录中的 `.bashrc` 文件。

>  `.bashrc`文件有两个作用：一是查看 /etc 目录下通用的`bashrc`文件，二是为用户提供一个定制自己的命令别名 和私有脚本函数的地方。



#### 5.1.3 非交互式shell

> 系统执行shell脚本时用的就是这种shell。不同的地方在于它没有命令行提示符。但是当你在系统上运行脚本时，也许希望能够运行一些特定启动的命令。



BASH_ENV环境变量：

> 为了处理这种情况，bashshell提供了BASH_ENV环境变量。当shell启动一个非交互式shell进程时，它会检查这个环境变量来查看要执行的启动文件。如果有指定的文件，shell会执行该文件里的命令，这通常包括shell脚本变量设置。





### 5.2 环境变量持久化

通过我们通过命令行对修改了环境变量，这种修改只能持续到退出或重启系统。

环境变量持久化修改，可通过如下方式：

-  修改 ` /etc/profile`
- 修改 `/etc/profile.d`



#### 5.2.1 修改 ` /etc/profile`

对全局环境变量来说（Linux系统中所有用户都需要使用的变量），可能更倾向于**将新的或修改过的变量设置放在/etc/profile文件中**，但这可不是什么好主意。如果你升级了所用的发行版，这个文件也会跟着更新，那你所有定制过的变量设置可就都没有了。



#### 5.2.2 修改 `/etc/profile.d`

最好是在 `/etc/profile.d` 目录中创建一个以.sh结尾的文件。把所有新的或修改过的全局环境变量设置放在这个文件中。

（1）与修改 ` /etc/profile`的区别

通过查看  ` /etc/profile ` 文件，我们会发现文件中包含如下语句:

```shell
## 遍历 /etc/profile.d/*.sh 文件
for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then 
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done

```



因此，我们可以发现其实两种方式的实现效果都一样，但是通过 `/etc/profile.d` 来设置会更好维护。



（2）如何设置环境变量

比如要设置 `Java`的环境变量：

- 在 `/etc/profile.d` 目录下创建 java.sh文件

```shell
sudo vim /etc/profile.d/java.sh
```



- 文件内容如下：

```shell
# java env
export JAVA_HOME=/opt/jdk1.8.0_192
export PATH=$JAVA_HOME/bin:$PATH
```











