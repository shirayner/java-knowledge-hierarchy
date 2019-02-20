[TOC]





# 前言





# 一、解决冲突

> 开发过程中要时刻保持和远程master代码一致

## 1.冲突解读

有时合并分支时，会与别人的代码产生冲突，如下所示：

```bash
$ git merge feature-zyj-hec-58
Auto-merging zyj-hec/src/main/resources/com/hand/hec/bgt/mapper/BgtOrganizationMapper.xml
CONFLICT (content): Merge conflict in zyj-hec/src/main/resources/com/hand/hec/bgt/mapper/BgtOrganizationMapper.xml
Auto-merging zyj-hec/src/main/java/com/hand/hec/bgt/service/impl/BgtOrganizationServiceImpl.java
CONFLICT (content): Merge conflict in zyj-hec/src/main/java/com/hand/hec/bgt/service/impl/BgtOrganizationServiceImpl.java
Auto-merging zyj-hec/src/main/java/com/hand/hec/bgt/service/IBgtOrganizationService.java
Automatic merge failed; fix conflicts and then commit the result.
```



> - Auto-merging  ：自动合并
> - CONFLICT (content) ：有内容冲突的文件



打开冲突文件，可以看到类似如下冲突内容

```java
<<<<<<< HEAD     //此处为当前分支
public static
=======
public void
>>>>>>> 6853e5ff961e684d3a6c02d4d06183b5ff330dcc
```



> - `<<<<<<< HEAD`  与 `=======` 之间的是当前分支内容
> -  `=======` 与 `>>>>>>> `分支名之间的是别的分支的内容



## 2.冲突解决

若拉取代码，或者推送代码时，提示有冲突（记住有冲突的文件），则可选择如下两种方法解决

（1）强制合并之后，一个一个解决（不推荐，自己百度）

（2）将自己的冲突文件（或自己所有修改过的文件）备份到任意一个地方，

然后将工作目录的代码回滚到和远程master相同（通常协作分支上才会有冲突，此过程请参考三），

然后将备份的冲突文件内容手动搬过来







# 参考资料

