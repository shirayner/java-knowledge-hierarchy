

[TOC]



# 前言

# 命令清单





# 一、合并远程分支到本地分支

## 1.`git fetch`

```bash
$ git fetch <远程主机名>   			  # 这个命令将某个远程主机的更新全部取回本地
$ git fetch <远程主机名> <分支名> 		# 注意之间有空格
```



示例：

```bash
$ git fetch origin master:temp        # 从远程的origin仓库的master分支下载到本地并新建一个分支temp

$ git diff temp                       # 比较master分支和temp分支的不同

$ git merge temp                      # 合并temp分支到master分支

$ git branch -d temp                  # 删除temp
```



## 2.`git pull`

```bash
$ git pull <远程主机名> <远程分支名>:<本地分支名>  # 取回远程主机某个分支的更新，再与本地的指定分支合并
```



如果远程分支是与当前分支合并，则冒号后面的部分可以省略：

```bash
$ git pull origin master  # 拉取远程master分支，并合并到本地当前分支
```











# 参考资料

1. [git fetch 合并远程仓库到本地](https://blog.csdn.net/lucyxu107/article/details/85275186)
2. [git fetch & pull详解](https://www.cnblogs.com/runnerjack/p/9342362.html)



