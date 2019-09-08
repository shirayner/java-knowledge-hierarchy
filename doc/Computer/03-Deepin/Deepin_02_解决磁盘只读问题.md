[TOC]





# 前言



# 一、Deepin异常汇总

## 1.解决磁盘只读问题

### 1.1 异常现象

（1）安装Deepin之前，盘符情况

> 原本系统win10装在C盘（win10系统盘），另有D(dev)、E(doc)盘。

（2）安装Deepin之后，盘符情况

> 接着，我用C盘覆盖安装了Deepin15.11系统，装好后系统盘符为一个系统 盘，两个本地磁盘
>
> ![img ](images/forum.php)
>
> 
>
> ![img](images/forum-1567858255097.php)



(3) 出现磁盘只读问题

>  我在doc盘中创建一个文件时，报出只读文件系统的错误
> ![img](images/forum-1567858294422.php)
>
> 意思就是，这两个本地磁盘我都用不了，只能读，不能写





### 1.2 异常解决

通常按照如下步骤即可解决

（1）执行`df -h` 查看磁盘挂载情况

```bash
ray@ray:~$ df -h
文件系统        容量  已用  可用 已用% 挂载点
udev            7.8G     0  7.8G    0% /dev
tmpfs           1.6G  1.7M  1.6G    1% /run
/dev/sdb4       219G   13G  195G    7% /
tmpfs           7.8G   31M  7.8G    1% /dev/shm
tmpfs           5.0M  4.0K  5.0M    1% /run/lock
tmpfs           7.8G     0  7.8G    0% /sys/fs/cgroup
/dev/sdb2        96M   33M   64M   35% /boot/efi
tmpfs           1.6G   20K  1.6G    1% /run/user/1000
/dev/sdc2       196G  156G   40G   80% /media/ray/doc
/dev/sdc1       271G  158G  113G   59% /media/ray/dev

```



（2）对挂载的只读磁盘（我的是 doc、dev 两个），执行如下命令

```bash
sudo ntfsfix /dev/sdc2
sudo ntfsfix /dev/sdc1
```

如果有多个NTFS磁盘都是只读，多次执行`ntfsfix`命令即可



（3）重启



（4）通常，执行前面三步即可解决问题。若仍无法解决，则重启之后，再做一遍前面三步，即可解决问题（我的问题就是这么解决的）。





# 参考资料

1. [deepin下挂载的的Windows系统（NTFC）目录怎么是只读的？？？](https://www.cnblogs.com/yulongcode/p/11246017.html)
2. [deepin下挂载的的Windows系统（NTFC）目录怎么是只读的？？？](https://bbs.deepin.org/forum.php?mod=viewthread&tid=140654)
3. 