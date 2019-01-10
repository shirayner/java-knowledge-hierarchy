[TOC]





# 前言

使用git命令时，保持脑海里有这幅图



![img ](images/UB94Upo.png)

> [本图来源](http://i.imgur.com/UB94Upo.png)



# 一、准备阶段

此阶段主要是将远程代码拉到自己本地，并切换到自己的分支上

## 1.拉取远程代码

```bash
git clone 
```



## 2.切换到自己的分支上

```bash
git checkout myBranch
```



# 二、开发过程中

多人协作时，若自己的本地代码有了修改，想提交自己的代码，就需要按照以下步骤操作：

- 检查有哪些修改
- 保证当前分支和远程master代码一致
- 提交自己的修改
- 

## 1.确认修改正确

使用以下命令，查看有哪些是自己未提交的代码

```
git status
```

 

## 2.保证当前分支和远程master代码一致

### 2.1 拉取远程master代码

```shell
git pull origin master
```

 

### 2.2 合并master到自己分支

执行如下命令，将master合并到当前分支

```shell
git merge master
```





## 3.提交自己的修改

### 3.1 提交到远程自己的分支上

```shell
git add --all
git commit -m"提交的信息"
git push origin myBranch
```

 

### 3.2 提交到远程master分支上

- 若没有权限，则提交合并请求

通常master分支是有被保护的，需要我们去提交合并请求，因此去猪齿鱼上提交一个合并请求。

合并请求通过之后，远程maste、 远程自己分支以及本地自己分支就是同步的了。



- 若有权限，则可以直接使用命令

```shell
git checkout master    # 切换到master分支
git merge  myBranch    # 合并自己本地分支上代码到本地master上,需要用户有master分支的权限
git push origin master

# 同步本地 master 到 远程master, 这时 本地自己分支、本地master分支、远程master分支、远程自己分支 就保持同步了
```







# 三、放弃自己本地代码

## 1.未使用 git add 缓存代码时

```
git checkout -- filepathname    //  放弃某个文件
git checkout .                  // 放弃所有文件
```


git checkout . 用来放弃掉所有还没有加入到缓存区（就是 git add 命令）的修改：内容修改与整个文件删除。

但是此命令不会删除掉刚新建的文件。因为刚新建的文件还没已有加入到 git 的管理系统中。所以对于git是未知的。自己手动删除就好了

 

## 2.已经使用 git add 缓存了代码

```
git reset HEAD filepathname  // 放弃指定文件的缓存
git reset HEAD .    // 放弃所有文件的缓存
```

 

此命令用来清除 git  对于文件修改的缓存。相当于撤销 git add 命令所在的工作。

在使用本命令后，本地的修改并不会消失，而是回到了如（一）所示的状态。继续用（一）中的操作，就可以放弃本地的修改。

 

## 3.已经用 git commit  提交了代码

```
git reset --hard HEAD^    // 回退到上一次commit的状态
git reset --hard  commitid   // 回退到任意版本
```

 

使用 git log 命令来查看git的提交历史，可以找到 commitid



## 4.放弃新建而没有被git管理的文件

删除当前目录下所有没有track过的文件. 不会删除.gitignore文件里面指定的文件夹和文件, 不管这些文件有没有被track过

```shell
git clean -f   
```



git reset --hard和git clean -f是一对好基友. 结合使用他们能让你的工作目录完全回退到最近一次commit的时候



# 四、解决冲突

> 开发过程中要时刻保持和远程master代码一致



若拉取代码，或者推送代码时，提示有冲突（记住有冲突的文件），则可选择如下两种方法解决

（1）强制合并之后，一个一个解决（不推荐，自己百度）

（2）将自己的冲突文件（或自己所有修改过的文件）备份到任意一个地方，

然后将工作目录的代码回滚到和远程master相同（通常协作分支上才会有冲突，此过程请参考三），

然后将备份的冲突文件内容手动搬过来





