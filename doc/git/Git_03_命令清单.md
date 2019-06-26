[TOC]



# 前言

在上一节 [Git_02_基本概念](./Git_02_基本概念.md) 中，我们提到下图，记住此图有利于我们理清git的相关命令



![img ](images/bg2015120901.png)









# 一、创建仓库

```bash
# 在当前目录初始化一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```





# 二、配置

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```bash
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config --global user.name "[name]"
$ git config --global user.email "[email address]"
```







# 三、代码提交

将代码提交到Git仓库的过程中，经历了如下区域：

> 工作目录 -> 暂存区 ->本地仓库



## 1.添加到暂存区

将代码添加到暂存区

```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加所有目录的中的已修改文件到暂存区
$ git add --all

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```





## 2.提交到本地仓库

> ```bash
> # 提交暂存区到仓库区
> $ git commit -m [message]
> 
> # 提交暂存区的指定文件到仓库区
> $ git commit [file1] [file2] ... -m [message]
> 
> # 提交工作区自上次commit之后的变化，直接到仓库区
> $ git commit -a
> 
> # 提交时显示所有diff信息
> $ git commit -v
> 
> # 使用一次新的commit，替代上一次提交
> # 如果代码没有任何新变化，则用来改写上一次commit的提交信息
> $ git commit --amend -m [message]
> 
> # 重做上一次commit，并包括指定文件的新变化
> $ git commit --amend [file1] [file2] ...
> ```



# 四、分支

## 1.查询分支

```bash
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a
```



## 2.创建分支

```bash
# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]
```



## 3.切换分支

```bash
# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]
```



## 4.本地关联远程分支

```bash
git branch --set-upstream-to=origin/remote_branch  local_branch
```



将本地分支与远程分支关联，这样在执行`git pull`, `git push`操作时就不需要指定对应的远程分支



## 5.合并分支

```bash
# 合并指定分支到当前分支
$ git merge [branch]
```



## 6.删除分支

```bash
# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```













# 五、标签

## 1.查询标签

```bash
# 列出所有tag
$ git tag


# 查看tag信息
$ git show [tag]
```



## 2.新建标签

```bash
# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]
```



## 3.删除标签

```bash
# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]
```



## 4.提交标签

```bash
# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags
```







# 六、查看信息

## 1.显示当前状态

```bash
# 显示有变更的文件
$ git status
```



## 2.版本历史

### 2.1 显示版本历史

```bash
# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature
```



### 2.3 显示指定文件的版本历史

```bash
# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]
```





### 2.4 其他

```bash
# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn
```



## 3.差异

### 3.1 暂存区与工作区的差异

```bash
# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]



# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"
```



### 3.2 工作区与本地仓库的差异

```bash
# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD
```





# 七、远程仓库

## 1.查询远程仓库

```bash
# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remoteName]
```



## 2.关联远程仓库

```bash
# 关联一个远程仓库，并命名
$ git remote add [remoteName] [url]


# 解除与远程仓库的关联
$ git remote rm [remoteName]
```



## 3.重命名远程主机名

```bash
$ git remote rename [oldRemoteName] [newRemoteName]
```







# 八、远程同步

## 1.拉取远程的更新

```bash
# 拉取远程所有分支的更新
$ git fetch <remoteName>   			 

# 拉取指定远程分支的更新
$ git fetch <remoteName>  <remoteBranchName> 	 	
```



所取回的更新，在本地主机上要用"远程主机名/分支名"的形式读取。比如`origin`主机的`master`，就要用`origin/master`读取。



例如：

```bash
# 拉取远程master分支的更新，在本机上要用`origin/master`读取
$ git fetch origin master

$ git branch -r
origin/master

$ git branch -a
* master
  remotes/origin/master
  
# 在当前分支上，合并origin/master
$ git merge origin/master
# 或者
$ git rebase origin/master
```







## 2.拉取并合并远程的更新

```bash
$ git pull <远程主机名>  <远程分支名>:<本地分支名>  # 取回远程主机某个分支的更新，再与本地的指定分支合并
```



如果远程分支是与当前分支合并，则冒号后面的部分可以省略

```bash
$ git pull origin next
```





## 3.推送代码到远程



> ```bash
> # 上传本地指定分支到远程仓库
> $ git push [remote] [branch]
> 
> # 强行推送当前分支到远程仓库，即使有冲突
> $ git push [remote] --force
> 
> # 推送所有分支到远程仓库
> $ git push [remote] --all
> ```



# 九、撤销

> ```bash
> # 恢复暂存区的指定文件到工作区
> $ git checkout [file]
> 
> # 恢复某个commit的指定文件到暂存区和工作区
> $ git checkout [commit] [file]
> 
> # 恢复暂存区的所有文件到工作区
> $ git checkout .
> 
> # 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
> $ git reset [file]
> 
> # 重置暂存区与工作区，与上一次commit保持一致
> $ git reset --hard
> 
> # 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
> $ git reset [commit]
> 
> # 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
> $ git reset --hard [commit]
> 
> # 重置当前HEAD为指定commit，但保持暂存区和工作区不变
> $ git reset --keep [commit]
> 
> # 新建一个commit，用来撤销指定commit
> # 后者的所有变化都将被前者抵消，并且应用到当前分支
> $ git revert [commit]
> 
> # 暂时将未提交的变化移除，稍后再移入
> $ git stash
> $ git stash pop
> ```



# 十、其他

> ```bash
> # 生成一个可供发布的压缩包
> $ git archive
> ```







# 参考资料

1. [阮一峰_常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
2. [阮一峰_Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
3. [Git 使用规范流程](http://www.ruanyifeng.com/blog/2015/08/git-use-process.html)