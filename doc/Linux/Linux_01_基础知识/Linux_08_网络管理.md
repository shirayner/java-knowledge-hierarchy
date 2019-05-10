[TOC]





# 前言







# 一、网络设置

## 1.默认网卡设备文件

文件路径：/etc/sysconfig/network-scripts/ifcfg-网卡名



前面设置静态ip的时候，修改了如下属性：

```shell
BOOTPROTO="static"   #设置网卡引导协议为 静态

ONBOOT="yes"         #设置网卡启动方式为 开机启动 并且可以通过系统服务管理器 systemctl 控制网卡

IPADDR="192.168.1.11"      # IP地址     
NETMASK="255.255.255.0"    # 子网掩码
GATEWAY="192.168.1.2"      # 网关,网关地址注意与前面查出来的保持一致，网关不对的话，后面是无法联网的

DNS1="114.114.114.114" 
DNS2="8.8.8.8"
```





## 2.DNS客户端配置

### 2.1 `/etc/hosts`

人们使用hosts文件来记录主机名和IP的对应关系，这样访问对方的主机时，就不需要使用IP了，只需要使用主机名。

这个文件在Linux下就是/etc/hosts，这种方式确实“可以工作”，但是**当主机数量增长到一定数量级的时候仍然无法适用**。为了彻底解决这个问题，人们发明了DNS系统。经过几十年的发展，虽然系统、网络技术都发生了翻天覆地的变化，但是这个文件还是被当作传统保留了下来。



具体来说，hosts文件的作用主要如下：

- **加快域名解析**。当访问网站时，系统会首先查看hosts文件中是否有记录，如果记录存在则直接解析出对应的IP，这时则不需要请求DNS服务器。 
- **方便小型局域网用户使用的内部设备**。很多单位的局域网中都存在着不少内部应用系统（比如办公自动化OA、公司论坛等），平时在工作中也都需要访问，但是由于这些局域网太小而不必为此专门设置DNS服务器，那么此时使用hosts文件则能简单地解决这个问题。



假设公司里有A、B两台主机，B主机的IP为`10.1.1.145`，为了方便访问B主机，可以在A主机的 `/etc/hosts`文件中添加一条记录：

>  10.1.1.145      hostB



### 2.2 `/etc/resolv.conf`

使用hosts文件毕竟只能做有限的主机记录，无法将所有已知的主机名记录到hosts文件中。

因此，当今几乎所有的主机都在使用DNS来解析地址，从技术上来说，**DNS就是全互联网上主机名及其IP地址对应关系的数据库**。设置主机为DNS客户端的配置文件就是 `/etc/resolv.conf`，其中包含 `nameserver`、`search`、`domain`这3个关键字。



以下是当前笔者测试机上的/etc/resolv.conf文件：

```shell
[root@localhost ~]# cat /etc/resolv.conf 
# generated by /sbin/dhclient-script 
search localdomain 
nameserver 192.168.159.2
```



nameserver关键字后面紧跟着一个DNS主机的IP地址，可以设置2~3个nameserver，

但是主机在查询域名时会首先查询第一个DNS，当该DNS不可用时才会查询第二个DNS，以此类推。







# 二、网络测试工具

## 1.ping

ping程序的目的在于测试另一台主机是否可达:

>  一般来说，如果ping不到某台主机，就说明对方主机已经出现了问题，但是不排除由于链路中防火墙的因素、ping包被丢弃等原因而造成ping不通的情况。



```shell
ping  IP/HOST
```





example:

```shell
ping  10.212.121.12
ping  www.baidu.com
```





## 2.host

host命令是用来查询DNS记录的，如果使用域名作为host的参数，命令返回该域名的IP，如下所示：

```shell
[ray@localhost dev]$ host www.baidu.com
www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 14.215.177.39
www.a.shifen.com has address 14.215.178.60
www.a.shifen.com has address 220.181.112.147
www.a.shifen.com has address 220.181.111.111
www.a.shifen.com has address 14.215.177.38
www.a.shifen.com has address 180.97.33.108
www.a.shifen.com has address 180.97.33.107
www.a.shifen.com has address 220.181.112.244
www.a.shifen.com has address 14.215.178.61
www.a.shifen.com has address 220.181.111.149
```



# 三、网络故障解决

## 1.故障排除步骤

（1）确认网卡本身是否能正常工作

> 利用ping工具可以确认这点。输入ping 127.0.0.1，然后看是否能正常ping通？这里的127.0.0.1被称为主机的回环接口，是TCP/IP协议栈正常工作的前提。如果ping不通，一般可以证实为本机TCP/IP协议栈有问题，自然就无法连接网络了。不过，出现这种现象的概率比较低。



（2）确认网卡是否出现了物理或驱动故障

> 使用ping本机IP地址的方式，如果能ping通则说明本地设备和驱动都正常。 



（3）确认是否能ping通同网段的其他主机

> 这一步主要是确认二层网络设备（比如交换机或者HUB）工作是否正常。如果ping不通往往说明二层网络上出现了问题，可能涉及交换机的端口工作模式、vlan划分等因素。 



（4）确认是否能ping通网关IP

> 如果数据包能正常到达网关，则说明主机和本地网络都工作正常。 



（5）确认是否能ping通公网上的IP

> 如果可以则说明本地的路由设置正确，否则就要确认路由设备是否做了正确的nat或路由设置。



（6）确认是否能ping通公网上的某个域名

> 如果能ping通则说明DNS部分设置正确。 



即便实际工作中可能会受到诸如更复杂的网络环境、安全ACL、防火墙等众多因素的影响，而加大了网络排查的困难，但以上步骤是排除网络故障的主要环节，在排除不同的网络之间个性化的设置之后，排查的主要步骤都与此类似。

