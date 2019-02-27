[TOC]



# 前言







# 一、创建用户

## 1.创建用户

```shell
useradd ray
```

## 2.设置密码

```shell
# 设置密码，设置密码必须在root用户下 
passwd ray   
```







# 二、授权

## 1.授予sudo权限

新创建的用户并不能使用sudo命令，需要给他添加授权。



查看`/etc/sudoers`文件，可看到如下部分：

```shell
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software, 
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

```



可知，可通过如下方式授予sudo权限：

- 加入 wheel 组
- 修改sudoers文件







### 1.1 加入 wheel 组

```shell
usermod -aG wheel ray
```





### 1.2 修改sudoers文件



**（1）添加sudoers文件可写权限**

```shell
# 查找sudoers文件位置
[root@ray opt]# whereis sudoers 
sudoers: /etc/sudoers /etc/sudoers.d /usr/share/man/man5/sudoers.5.gz

# 查看权限
[root@ray opt]# ls -l /etc/sudoers     
-r--r----- 1 root root 4328 Oct 30 22:38 /etc/sudoers

# 赋予读写权限
[root@ray opt]# chmod -v u+w /etc/sudoers
mode of ‘/etc/sudoers’ changed from 0440 (r--r-----) to 0640 (rw-r-----)


```



**（2）修改sudoers文件**



```shell
# 修改 sudoers 文件
vim /etc/sudoers

# 添加新用户信息
## Allow root to run any commands anywhere 
root ALL=(ALL) ALL
ray  ALL=(ALL) ALL  #这个是新用户
```



**（3）收回权限**

```shell
chmod -v u-w /etc/sudoers
```







