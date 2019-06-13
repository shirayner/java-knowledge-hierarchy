[TOC]





# 前言





# 一、为镜像添加SSH

## 1.基于 commit 命令创建



### 1.1 准备工作

（1）安装操作系统

```bash
docker run --name ray-ubuntu -it ubuntu /bin/bash
```



（2）更新apt（包管理工具）缓存，并安装openssh-server

```
apt-get update
apt-get install openssh-server -y
```



### 1.2 安装和配置SSH服务

#### 1.2.1 启动SSH服务

如果需要正常启动SSH服务，则目录`/var/run/sshd`必须存在。手动创建它，并启动SSH服务：

```bash
mkdir -p /var/run/sshd
/usr/sbin/sshd -D &
```



此时查看容器的22端口（SSH服务默认监听的端口），可见此端口已经处于监听状态：

```bash
netstat -tunlp
```

当前版本的 ubuntu 对命令进行了精简，若出现`bash: netstat: command not found`，则需要安装 netstat 命令，然后再执行即可

```
 apt-get install net-tools -y
```



#### 1.2.2 取消pam登录限制

修改SSH服务的安全登录配置，取消pam登录限制：

```bash
sed -ri 's/session    required    pam_loginuid.so/#session required    pam_loginuid.so/g' /etc/pam.d/sshd
```





#### 1.2.2 添加SSH公钥

在root用户目录下创建`.ssh`目录，并复制需要登录的公钥信息（一般为本地主机用户目录下的 `.ssh/id_rsa.pub`文件，可由`ssh-keygen-trsa`命令生成）到`authorized_keys`文件中：

```
mkdir ~/.ssh
vim  ~/.ssh/authorized_keys
```



#### 1.2.3 创建SSH服务自启动脚本

创建自动启动SSH服务的可执行文件run.sh，

```
vim /run.sh
```

内容如下：

```
#!/bin/bash 
/usr/sbin/sshd -D
```



并添加可执行权限：

```
chmod +x run.sh
```



最后退出容器



#### 1.2.4 保存镜像

我们将 ray-ubuntu 容器保存为 shirayner/ray-ubuntu 镜像

```
docker commit -m"added ssh" ray-ubuntu shirayner/ray-ubuntu
```

查看镜像：

```bash
ray@ray:/mnt/d/docker-repository/docker-in-action/in-action$ docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
shirayner/ray-ubuntu   latest              21678e803a65        7 seconds ago       246MB
ubuntu                 latest              7698f282e524        2 weeks ago         69.9MB
```



### 1.2.5 使用镜像

启动容器，并添加端口映射10022-->22。其中10022是宿主主机的端口，22是容器的SSH服务监听端口：

```
docker run -p 10022:22 -d  shirayner/ray-ubuntu  /run.sh
```



在宿主主机（101.231.252.98）或其他主机上上，可以通过SSH访问10022端口来登录容器：

```
ssh 101.231.252.98 -p 10022
```



## 2.使用Dockerfile创建













# 参考资料





