[TOC]





# 前言

使用git命令时，保持脑海里有这幅图



![img ](images/UB94Upo.png)

> [本图来源](http://i.imgur.com/UB94Upo.png)





# 一、提交自己的修改

多人协作时，若自己的本地代码有了修改，想提交自己的代码，就需要按照以下步骤操作：

- 保存自己的修改
- 合并远程 `master` 分支代码
- 提交修改到 `master` 分支



## 1.准备阶段

此阶段主要是将远程代码拉到自己本地，并切换到自己的分支上

### 1.1拉取远程代码

```bash
git clone 
```



### 1.2切换到自己的分支上

```bash
git checkout myBranch
```

然后我们就可以在自己的分支上进行开发了



## 2.保存自己的修改

使用以下命令，查看有哪些是自己本次修改而未提交的代码

```bash
git status
```



提交修改到远程自己的分支上

```bash
git add --all                 # 提交至暂存区
git commit -m"提交的信息"       # 提交至本地仓库
git push origin myBranch      #提交至远程仓库
```



## 3.合并远程 `master` 分支代码

使用如下命令，拉取远程 master 分支代码，并合并到本地当前分支上，这样就保证本地与远程`master`代码同步

```bash
git pull origin master
```





## 4.提交修改到 `master` 分支

使用如下命令，将本地  `myBranch` 分支的代码提交至远程`master`分支上，这样就保证了远程`master`分支与本地分支代码同步

```
git push origin myBranch:master
```



至此远程`master`代码与本地当前分支代码一致了。





# 三、放弃自己的修改



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

```java
<<<<<<< HEAD     //此处为当前分支
public static
=======
public void
>>>>>>> 6853e5ff961e684d3a6c02d4d06183b5ff330dcc
```

<<<<<<< HEAD  与 ==== 之间的是 当前分支内容，   =======  与 >>>>>>> 分支名   是别的分支的内容

