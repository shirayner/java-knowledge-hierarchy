[TOC]



# 前言

本文转自 [菜鸟教程_Docker命令大全](https://www.runoob.com/docker/docker-command-manual.html)





所有命令的选项参数可通过 添加`--help`来查询，如：

```bash
docker pull --help
```



# 一、镜像

## 1.镜像仓库

### 1.1 login/logout

> - **docker login :** 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub
> - **docker logout :** 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub



**语法**：

```bash
docker login [OPTIONS] [SERVER]
docker logout [OPTIONS] [SERVER]
```



OPTIONS说明：

> - **-u :**登陆的用户名
> - **-p :**登陆的密码



**实例**：

（1）登入登出 Docker Hub

```bash
docker login -u 用户名 -p 密码
docker logout
```

（2）登入登出阿里云Docker Hub Mirror 

```bash
docker login -u shiray1994@163.com registry.cn-shanghai.aliyuncs.com
docker logout
```



### 1.2 search

> **docker search :** 从Docker Hub查找镜像



**语法**：

````bash
docker search [OPTIONS] IMAGE
````



OPTIONS说明：

> - **--automated :**只列出 automated build类型的镜像；
> - **--no-trunc :**显示完整的镜像描述；
> - **-s :**列出收藏数不小于指定值的镜像。



**实例**：

从Docker Hub查找所有镜像名包含java，并且收藏数大于10的镜像

```bash
$ docker search -s 10 java
```



### 1.3 pull

> **docker pull :** 从镜像仓库中拉取或者更新指定镜像



**语法**

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```



OPTIONS说明：

> - **-a :**拉取所有 tagged 镜像
> - **--disable-content-trust :**忽略镜像的校验,默认开启



**实例**

从Docker Hub下载java最新版镜像。

```bash
docker pull java
```





**注意事项**：

```bash
$ docker pull  镜像名
```

注意这里的镜像名格式如下，主要包括三个部分：Docker镜像仓库地址、仓库名、标签

```
[Docker Registry 地址[:端口号]/]仓库名[:标签]
```

> - Docker 镜像仓库地址
>     - 地址的格式一般是 `<域名/IP>[:端口号]`。
>     - 默认地址是 Docker Hub。若要拉取或上传镜像到其他镜像仓库，则需要指定镜像仓库地址。
> - 仓库名
>     - 仓库名是两段式名称，即 `<用户名>/<软件名>`。
>     - 对于 Docker Hub，如果不给出用户名，则默认为 `library`，也就是官方镜像。
> - 标签
>     - 可通过标签来指定镜像的版本
>     - 若不指定标签，则默认为 lastest





### 1.4 push

> **docker push :** 将本地的镜像上传到镜像仓库,要先登陆到镜像仓库



**语法**

```
docker push [OPTIONS] NAME[:TAG]
```

OPTIONS说明：

> - **--disable-content-trust :**忽略镜像的校验,默认开启



**实例**

上传本地镜像 mysql 到镜像仓库中。

```bash
docker push shirayner/mysql:v1
```











## 2.本地镜像管理

### 2.1 images

> **docker images :** 列出本地镜像。



**语法**：

```bash
docker images [OPTIONS] [REPOSITORY[:TAG]]
```



OPTIONS说明：

> - **-a :**列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；
>
>     
>
> - **--digests :**显示镜像的摘要信息；
>
>     
>
> - **-f :**显示满足条件的镜像；
>
>     
>
> - **--format :**指定返回值的模板文件；
>
>     
>
> - **--no-trunc :**显示完整的镜像信息；
>
>     
>
> - **-q :**只显示镜像ID。



**实例**：

（1）列出所有镜像

```bash
docker images
```

（2）删除所有镜像（先查出所有的镜像的id列表，然后再据此删除镜像）

```bash
docker rmi $(docker images -aq)
```





### 2.2 rmi

> **docker rmi :** 删除本地一个或多少镜像。



**语法**：

```bash
docker rmi [OPTIONS] IMAGE [IMAGE...]
```



OPTIONS说明：

> - **-f :**强制删除；
> - **--no-prune :**不移除该镜像的过程镜像，默认移除；



**实例**：

（1）强制删除指定的本地镜像

```bash
docker rmi -f runoob/ubuntu:v4
```

（2）删除所有镜像（先查出所有的镜像的id列表，然后再据此删除镜像）

```bash
docker rmi $(docker images -aq)
```





### 2.3 tag

> **docker tag :** 标记本地镜像，将其归入某一仓库。打成符合的标记后，才可 push 至指定的仓库。



语法：

```bash
docker tag [OPTIONS]  IMAGE[:TAG]  [REGISTRYHOST/][USERNAME/]NAME[:TAG]
```



实例：

（1）给 ubuntu:15.10 镜像打标签，并上传至自己的 Docker Hub 的仓库下

```bash
docker  tag  ubuntu:15.10  shirayner/ubuntu:v3
docker push shirayner/ubuntu:v3
```

（2）给 ubuntu:15.10 镜像打标签，并上传至阿里云Docker Hub Mirror （或者自己搭建的Registry）自己的仓库下

```bash
$ sudo docker login -u shiray1994@163.com registry.cn-shanghai.aliyuncs.com
$ sudo docker tag ubuntu:15.10 registry.cn-shanghai.aliyuncs.com/shirayner/ubuntu:v3
$ sudo docker push registry.cn-shanghai.aliyuncs.com/shirayner/ubuntu:v3
```





### 2.4 build

> **docker build** 命令用于使用 Dockerfile 创建镜像。



**语法**：

```bash
docker build [OPTIONS] PATH | URL | -
```



> - docker build 从 Dockerfile 和上下文构建构建镜像。
>
> - Dockerfile的位置可以使：本地资源路径（PATH）、Git仓库地址(URL)



OPTIONS说明：

> - **--build-arg=[] :**设置镜像创建时的变量；
> - **--cpu-shares :**设置 cpu 使用权重；
> - **--cpu-period :**限制 CPU CFS周期；
> - **--cpu-quota :**限制 CPU CFS配额；
> - **--cpuset-cpus :**指定使用的CPU id；
> - **--cpuset-mems :**指定使用的内存 id；
> - **--disable-content-trust :**忽略校验，默认开启；
> - **-f :**指定要使用的Dockerfile路径；
> - **--force-rm :**设置镜像过程中删除中间容器；
> - **--isolation :**使用容器隔离技术；
> - **--label=[] :**设置镜像使用的元数据；
> - **-m :**设置内存最大值；
> - **--memory-swap :**设置Swap的最大值为内存+swap，"-1"表示不限swap；
> - **--no-cache :**创建镜像的过程不使用缓存；
> - **--pull :**尝试去更新镜像的新版本；
> - **--quiet, -q :**安静模式，成功后只输出镜像 ID；
> - **--rm :**设置镜像成功后删除中间容器；
> - **--shm-size :**设置/dev/shm的大小，默认值是64M；
> - **--ulimit :**Ulimit配置。
> - **--tag, -t:** 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。
> - **--network:** 默认 default。在构建期间设置RUN指令的网络模式





实例：

（1）使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1

```
docker build -t runoob/ubuntu:v1 . 
```

（2）使用URL ` github.com/creack/docker-firefox` 的 Dockerfile 创建镜像。

```
docker build github.com/creack/docker-firefox
```

（3）也可以通过 -f Dockerfile 文件的位置：

```
docker build -f /path/to/a/Dockerfile .
```





### 2.5 history

> **docker history :** 查看指定镜像的创建历史。



**语法**：

```bash
docker history [OPTIONS] IMAGE
```



OPTIONS说明：

> - **-H :**以可读的格式打印镜像大小和日期，默认为true；
> - **--no-trunc :**显示完整的提交记录；
> - **-q :**仅列出提交记录ID。



**实例**：

（1）查看本地镜像runoob/ubuntu:v3的创建历史。

```
docker history runoob/ubuntu:v3
```





### 2.6 save

> **docker save :** 将指定镜像保存成 tar 归档文件。



语法：

```bash
docker save [OPTIONS] IMAGE [IMAGE...]
```



OPTIONS 说明：

> **-o :**输出到的文件。



实例：

（1）将镜像 runoob/ubuntu:v3 生成 my_ubuntu_v3.tar 文档

```bash
 docker save -o my_ubuntu_v3.tar runoob/ubuntu:v3
```



### 2.7 load

> **docker load :** 导入使用 docker save 命令导出的镜像。



**语法**：

```
docker load [OPTIONS]
```

OPTIONS 说明：

> - **-i :** 指定导出的文件。
> - **-q :** 精简输出信息。



**实例**：

（1）导入镜像：

```bash
docker load -i ubuntu.tar
docker load < ubuntu.tar
```



### 2.8 import 

> **docker import :** 从归档文件中创建镜像。



语法：

```bash
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```



OPTIONS说明：

> - **-c :**应用docker 指令创建镜像；
> - **-m :**提交时的说明文字；



实例：

从镜像归档文件my_ubuntu_v3.tar创建镜像，命名为runoob/ubuntu:v4

```
docker import  my_ubuntu_v3.tar runoob/ubuntu:v4  
```





# 二、容器

## 1. 容器生命周期管理

### 1.1 run

> **docker run ：**创建一个新的容器并运行一个命令



语法：

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```



OPTIONS说明：

> - **-a stdin:** 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；
> - **-d:** 后台运行容器，并返回容器ID；
> - **-i:** 以交互模式运行容器，通常与 -t 同时使用；
> - **-P:** 随机端口映射，容器内部端口**随机**映射到主机的高端口
> - **-p:** 指定端口映射，格式为：**主机(宿主)端口:容器端口**
> - **-t:** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
> - **--name="nginx-lb":** 为容器指定一个名称；
> - **--dns 8.8.8.8:** 指定容器使用的DNS服务器，默认和宿主一致；
> - **--dns-search example.com:** 指定容器DNS搜索域名，默认和宿主一致；
> - **-h "mars":** 指定容器的hostname；
> - **-e username="ritchie":** 设置环境变量；
> - **--env-file=[]:** 从指定文件读入环境变量；
> - **--cpuset="0-2" or --cpuset="0,1,2":** 绑定容器到指定CPU运行；
> - **-m :**设置容器使用内存最大值；
> - **--net="bridge":** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
> - **--link=[]:** 添加链接到另一个容器；
> - **--expose=[]:** 开放一个端口或一组端口；



**实例**：

（1）使用docker镜像nginx:latest以后台模式启动一个容器,并将容器命名为mynginx。

```bash
docker run --name mynginx -d nginx:latest
```

（2）使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。

```bash
docker run -p 80:80 -v /data:/data -d nginx:latest
```

（3）绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。

```bash
 docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
```





### 1.2 start/stop/restart

> - **docker start** : 启动一个或多个已经被停止的容器
>
> - **docker stop** : 停止一个运行中的容器
>
> - **docker restart** : 重启容器



语法：

```bash
docker start [OPTIONS] CONTAINER [CONTAINER...]
docker stop [OPTIONS] CONTAINER [CONTAINER...]
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```



实例：

（1）启动已被停止的容器 mysql

```bash
docker start mysql
```

（2）停止运行中的容器 mysql

```bash
docker stop mysql
```

（3）重启容器 mysql

```bash
docker restart mysql
```





### 1.3 kill 

> **docker kill** :杀掉一个运行中的容器。



**语法**：

```bash
docker kill [OPTIONS] CONTAINER [CONTAINER...]
```



OPTIONS说明：

> **-s :**向容器发送一个信号



**实例**：

（1）杀掉运行中的容器 nginx

```bash
docker kill -s KILL nginx
```





### 1.4 rm 

> **docker rm ：**删除一个或多少容器



语法：

```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```



OPTIONS说明：

> - **-f :**通过SIGKILL信号强制删除一个运行中的容器
> - **-l :**移除容器间的网络连接，而非容器本身
> - **-v :**-v 删除与容器关联的卷



实例：

（1）强制删除容器db01、db02

```bash
docker rm -f db01 db02
```

（2）移除容器nginx01对容器db01的连接，连接名db

```bash
docker rm -l db 
```

（3）删除容器nginx01,并删除容器挂载的数据卷

```bash
docker rm -v nginx01
```



### 1.5 pause/unpause

> - **docker pause** :暂停容器中所有的进程。
> - **docker unpause** :恢复容器中所有的进程。



语法：

```bash
docker pause [OPTIONS] CONTAINER [CONTAINER...]
docker unpause [OPTIONS] CONTAINER [CONTAINER...]
```



实例：

（1）暂停数据库容器db01提供服务。

```bash
docker pause db01
```

（2）恢复数据库容器db01提供服务。

```bash
docker unpause db01
```



### 1.6 create 

> **docker create ：**创建一个新的容器但不启动它



语法：

```bash
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

语法同 docker run



实例：

（1）使用docker镜像nginx:latest创建一个容器,并将容器命名为myrunoob

```bash
docker create  --name myrunoob  nginx:latest      
```



### 1.7 exec

> **docker exec ：**在运行的容器中执行命令



语法：

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```



OPTIONS说明：

> - **-d :**分离模式: 在后台运行
> - **-i :**即使没有附加也保持STDIN 打开
> - **-t :**分配一个伪终端



实例：

（1）在容器 mynginx 中以交互模式执行容器内 /root/runoob.sh 脚本:

```bash
docker exec -it mynginx /bin/sh /root/runoob.sh
```

（2）在容器 mynginx 中开启一个交互模式的终端:

```bash
docker exec -i -t  mynginx /bin/bash
```



也可以通过 **docker ps -a** 命令查看已经在运行的容器，然后使用容器 ID 进入容器。





## 2.容器操作

### 2.1 ps

> **docker ps :** 列出容器



**语法**：

```
docker ps [OPTIONS]
```



OPTIONS说明：

> - **-a :**显示所有的容器，包括未运行的。
> - **-f :**根据条件过滤显示的内容。
> - **--format :**指定返回值的模板文件。
> - **-l :**显示最近创建的容器。
> - **-n :**列出最近创建的n个容器。
> - **--no-trunc :**不截断输出。
> - **-q :**静默模式，只显示容器编号。
> - **-s :**显示总的文件大小。





实例：

（1）列出所有在运行的容器信息。

```
docker ps
```

（2）列出最近创建的5个容器信息。

```bash
docker ps -n 5
```

（3）列出所有创建的容器ID。

```bash
docker ps -a -q
```



### 2.2 inspect

> **docker inspect :** 获取容器/镜像的元数据。



**语法**：

```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```



OPTIONS说明：

> - **-f :**指定返回值的模板文件。
> - **-s :**显示总的文件大小。
> - **--type :**为指定类型返回JSON。



**实例**：

（1）获取镜像mysql:5.6的元信息。

```bash
docker inspect mysql:5.6
```



### 2.3 top

>  **docker top :**查看容器中运行的进程信息，支持 ps 命令参数。



语法：

```bash
docker top [OPTIONS] CONTAINER [ps OPTIONS]
```

容器运行时不一定有 /bin/bash 终端来交互执行top命令，而且容器还不一定有top命令，可以使用docker top来实现查看container中正在运行的进程。





实例：

（1）查看容器mymysql的进程信息。

```
docker top mymysql
```



### 2.4 attach

> **docker attach :**连接到正在运行中的容器。



语法：

```
docker attach [OPTIONS] CONTAINER
```

attach可以带上--sig-proxy=false来确保CTRL-D或CTRL-C不会关闭容器。



实例：

（1）容器mynginx将访问日志指到标准输出，连接到容器查看访问信息。

```bash
docker attach --sig-proxy=false mynginx
```





### 2.5 events

> **docker events :** 从服务器获取实时事件



语法：

```bash
docker events [OPTIONS]
```



OPTIONS说明：

> - **-f ：**根据条件过滤事件；
> - **--since ：**从指定的时间戳后显示所有事件;
> - **--until ：**流水时间显示到指定的时间为止；



实例：

（1）显示docker 镜像为mysql:5.6 2016年7月1日后的相关事件。

```
ocker events -f "image"="mysql:5.6" --since="1467302400" 
```





### 2.6 logs

> **docker logs :** 获取容器的日志



语法：

```bash
docker logs [OPTIONS] CONTAINER
```



OPTIONS说明：

> - **-f :** 跟踪日志输出
> - **--since :**显示某个开始时间的所有日志
> - **-t :** 显示时间戳
> - **--tail :**仅列出最新N条容器日志



实例：

（1）跟踪查看容器mynginx的日志输出。

```bash
docker logs -f mynginx
```

（2）查看容器mynginx从2016年7月1日后的最新10条日志。

```bash
docker logs --since="2016-07-01" --tail=10 mynginx
```



### 2.7 wait

> **docker wait :** 阻塞运行直到容器停止，然后打印出它的退出代码。



语法：

```bash
docker wait [OPTIONS] CONTAINER [CONTAINER...]
```



实例：

````
docker wait CONTAINER
````



### 2.8 export

> **docker export :**将文件系统作为一个tar归档文件导出到STDOUT。



语法：

```bash
docker export [OPTIONS] CONTAINER
```



OPTIONS说明：

> **-o :**将输入内容写到文件。



实例：

（1）将id为a404c6c174a2的容器按日期保存为tar文件。

```
docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2
```



### 2.9 port

> **docker port :**列出指定的容器的端口映射，或者查找将PRIVATE_PORT NAT到面向公众的端口。



语法：

```bash
docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]
```



实例：

（1）查看容器mynginx的端口映射情况。

```bash
runoob@runoob:~$ docker port mymysql
3306/tcp -> 0.0.0.0:3306
```



## 3.容器rootfs命令

### 3.1 commit

> **docker commit :**从容器创建一个新的镜像。



语法：

````bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
````



OPTIONS说明：

> - **-a :**提交的镜像作者；
> - **-c :**使用Dockerfile指令来创建镜像；
> - **-m :**提交时的说明文字；
> - **-p :**在commit时，将容器暂停。



实例：

（1）将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。

```bash
docker commit -a "runoob.com" -m "my apache" a404c6c174a2  mymysql:v1 
```





### 3.2 cp

> **docker cp :**用于容器与主机之间的数据拷贝。



语法：

```bash
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```



OPTIONS说明：

> **-L :**保持源目标中的链接





实例：

（1）将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。

```bash
docker cp /www/runoob 96f7f14e99ab:/www/
```

（2）将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。

```bash
docker cp /www/runoob 96f7f14e99ab:/www
```

（3）将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。

```bash
docker cp  96f7f14e99ab:/www /tmp/
```



### 3.3 diff

> **docker diff :** 检查容器里文件结构的更改



语法：

```bash
docker diff [OPTIONS] CONTAINER
```



实例：

（1）查看容器mymysql的文件结构更改。

```bash
docker diff mymysql
```















# 参考资料

1. [gitbook__Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)
2. [Docker 命令大全](https://www.runoob.com/docker/docker-command-manual.html)
3. [docker commandline](https://docs.docker.com/engine/reference/commandline/docker/)



