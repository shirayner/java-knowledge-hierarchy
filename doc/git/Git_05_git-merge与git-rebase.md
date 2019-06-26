[TOC]





# 前言

本文参考自：

> - [3.6 Git 分支 - 变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)
> - [git-rebase](https://git-scm.com/docs/git-rebase)



在 Git 中整合来自不同分支的修改主要有两种方法：`merge` 以及 `rebase`



我们先来接受一个观点：**每一次提交就会保存一个快照，而分支其实就是一个指向快照的指针**



# 一、merge

> git-merge - Join two or more development histories together
>
> 将其他分支的提交历史合并到当前分支



假设有如下提交历史：

![ ](images/basic-branching-6.png)



我们要合并master上修改

```bash
git checkout master
git merge iss53
```

这时，Git 会使用两个分支的末端所指的快照（`C4` 和 `C5`）以及这两个分支的工作祖先（`C2`），做一个简单的三方合并。



最后形成以下结果，实现提交的合并

![ ](images/basic-merging-2.png)





# 二、rebase

## 1.rebase原理

[git官网文档](https://git-scm.com/docs/git-rebase)给出的定义

> git-rebase - Reapply commits on top of another base tip
>
> 即在另一个基点上重新执行提交







假设有如下提交历史：

```
          A---B---C topic
         /
    D---E---F---G master
```



然后我们执行 rebase 命令

```
git rebase master
```

则git将执行如下操作：

> （1）首先找到这两个分支的最近共同祖先 `E`
>
> （2）然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件
>
> （3）接着将当前分支（topic指针）指向目标基底 `G`
>
> （4）将临时文件中保存的提交重新应用一遍

最后形成以下结果，实现提交的合并

```
                  A'--B'--C' topic
                 /
    D---E---F---G master
```





## 2.rebase使用场景

### 2.1 整合分支

假设有如下提交历史：

```
          A---B---C topic
         /
    D---E---F---G master
```



我们正处于topic分支，要合并master上的更新

```
git reabse master
```

最后形成以下结果，实现提交的合并

```
                  A'--B'--C' topic
                 /
    D---E---F---G master
```



### 2.2 合并多个commit

假设有如下提交历史：

```
          A---B---C topic
         /
    D---E---F---G master
```



我们正处于topic分支，要合并master上的更新

```
git reabse -i master
```

会进入vim编辑器，我们将commit都合并到A中，形成新的commit D'

```
p A
s B
s C
```

最后形成以下结果，实现提交的合并

```
                  D' topic
                 /
    D---E---F---G master
```



### 2.3 git pull

> - git pull =  git fetch + git merge
> - git pull --rebase = git fetch + git rebase













# 参考资料

1. [rebase](http://gitbook.liuhui998.com/4_2.html)
2. [git Merge 原理算法文章标题](https://blog.csdn.net/u012937029/article/details/77161584)
3. [这一次彻底搞懂 Git Rebase](https://www.codercto.com/a/45325.html)
4. [Rebase](https://www.liaoxuefeng.com/wiki/896043488029600/1216289527823648)
5. [【Git】rebase 用法小结](https://www.jianshu.com/p/4a8f4af4e803)
6. [Git 使用规范流程](https://www.jianshu.com/p/4a8f4af4e803)
7. [Git 之 交互式 rebase](https://segmentfault.com/a/1190000012897755)
8. [关于git rebase 的使用场景总结](https://juejin.im/entry/5cbfedffe51d456e831f6946#comment)