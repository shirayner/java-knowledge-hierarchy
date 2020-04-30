[toc]









# 前言

## 推荐阅读

> - [一个 Git 分支协作模式的进化故事](https://blog.gitee.com/2019/04/25/gitee-branch/)
> - [ 码云Git flow的最佳团队协作实践](https://gitee.com/oschina/gitee_best_practices/blob/master/%E9%A1%B9%E7%9B%AE%E7%AE%A1%E7%90%86/%E7%A0%81%E4%BA%91Git%20flow%E7%9A%84%E6%9C%80%E4%BD%B3%E5%9B%A2%E9%98%9F%E5%8D%8F%E4%BD%9C%E5%AE%9E%E8%B7%B5.md)
> - [如何写好 Git commit log?](https://www.zhihu.com/question/21209619/answer/257574960)
> - [优雅的提交你的 Git Commit Message](https://zhuanlan.zhihu.com/p/34223150)
> - [git commit 规范指南](https://segmentfault.com/a/1190000009048911)
> - [你可能会忽略的 Git 提交规范](https://blog.csdn.net/y491887095/article/details/80594043)
> - 









# 一、分支管理规范

目前比较流行的分支管理模型有三个，即`GitFlow`、`GitLabFlow`、`GitHubFlow`。下面将介绍这三种分支模型的原理，使用场景和优缺点等。

### GitFlow

`GitFlow` 是最早诞生并得到广泛应用的一种工作流程。

该模型中存在两种长期分支：`master` 和 `develop`。 `master`中存放对外发布的版本，只有稳定的发布版本才会合并到`master`中。 `develop`用于日常开发，存放最新的开发版本。

也存在三种临时分支：`feature`, `hotfix`, `release`。

- `feature`分支是为了开发某个特定功能，从`develop`分支中切出，开发完成后合并到`develop`分支中。
- `hotfix`分支是修复发布后发现的Bug的分支，从`master`分支中切出，修补完成后再合并到`master`和`develop`分支。
- `release`分支指发布稳定版本前使用的预发布分支，从`develop`分支中切出，预发布完成后，合并到`develop`和`master`分支中。

优点：

- `feature` 分支使开发代码隔离，可以独立的完成开发、构建、测试
- `feature` 分支开发周期长于`release`时，可避免未完成的`feature`进入生产环境

缺点：

- 无法支持持续发布。
- 过于复杂的分支管理，加重了开发者的负担，使开发者不能专注开发。

### GitHubFlow

`GitHubFlow`分支模型只存在一个`master`主分支，日常开发都合并至`master`，永远保持其为最新的代码且随时可发布的。

- 在需要添加或修改代码时， 基于`master`创建分支，提交修改。
- 创建`Pull Request`，所有人讨论和审查你的代码。
- 然后部署到生产环境中进行验证。
- 待验证通过后合并到`master`分支中。

这个分支模型的优势在于简洁易理解，将`master`作为核心的分支，代码更新持续集成至`master`上。根据目前收集到的反应来看，得到了更多的好评，认为`GitHubFlow`分支模型更加轻便快捷。

### GitLabFlow

`GitLabFlow` 是`GitFlow`和`GitHubFlow`的结合,它吸取了两者的优点，既有适应不同开发环境的弹性，又有单一主分支的简单和便利。

该模型采取上游优先的原则，即只存在一个`master`主分支，它是所以分支的上游。只有上游分支采纳的变动才能应用到其他分支。

- 对于持续发布的项目，建议在`master`之外再建立对应的环境分支，如预生产环境`pre-production`，生产环境`production`。
- 对于版本发布的项目，建议基于`master`创建稳定版本对应的分支，如`stable-1`，`stable-2`。

### Choerodon

Choerodon中采取了GitHubFlow的模式，并提供多种分支类型。

- 在领到日常开发任务时，基于`master`创建`feature`特性开发分支，提交代码后，合并至`master`并删除`feature`。
- 在领到修复故障的任务时，基于`master`创建`bugfix`故障修补分支，提交代码后，合并至`master`并删除`bugfix`。
- 需要发布时，同样需要基于`master`创建`release`，生成的应用版本部署在UAT测试环境进行测试，若需要修改则提交至`release`。
- 产品上线后发现故障需要紧急进行热修复时，则基于`tag`创建`hotfix`，将修复的代码提交至`hotfix`；部署该分支上的版本通过验收后，基于`hotfix`打出热修版本的`tag`，如`0.8.1`。
- 由于新版本的迭代也同时进行，所以需要在`hotfix`上`rebase master`，变基至`master`分支最新的提交，再合并至`master`并删除`hotfix`，就可以将本次修改的提交应用至`master`上。

### 分支命名规约

| 前缀       | 含义                                     |
| :--------- | :--------------------------------------- |
| master     | 主分支，可用的、稳定的、可直接发布的版本 |
| develop    | 开发主分支，最新的代码分支               |
| feature-** | 功能开发分支                             |
| bugfix-**  | 未发布bug修复分支                        |
| release-** | 预发布分支                               |
| hotfix-**  | 已发布bug修复分支                        |









# 二、提交命名规约

除了分支的名称需要规范，提交的命名也同样如此。猪齿鱼并没有把这个规则固化到系统中，需要团队共同遵守。

格式为：[操作类型]操作对象名称，如[ADD]readme，代表增加了readme描述文件。

常见的操作类型有：

- [IMP] 提升改善正在开发或者已经实现的功能
- [FIX] 修正BUG
- [REF] 重构一个功能，对功能重写
- [ADD] 添加实现新功能
- [REM] 删除不需要的文件
- [WIP] work in process
- [TEST] 新增或修改测试用例
- [STYLE] style 代码格式化改动，缺少分号等





## 三、工具的使用

工具列表：

> - [commitizen/cz-cli](https://link.zhihu.com/?target=https%3A//github.com/commitizen/cz-cli) 
> - [commitizen/cz-conventional-changelog](https://link.zhihu.com/?target=https%3A//github.com/commitizen/cz-conventional-changelog) 
> -  [conventional-changelog/standard-version](https://link.zhihu.com/?target=https%3A//github.com/conventional-changelog/standard-version)
> - [marionebl/commitlint](https://link.zhihu.com/?target=https%3A//github.com/marionebl/commitlint)



使用教程：

> - [Java项目如何使用commitizen插件](https://www.jianshu.com/p/dc3581887cbb)











# 参考资料

> - [Git提交日志格式规约](https://blog.csdn.net/Jason847/article/details/78617067)
> - [Git 提交日志格式规约](http://researchlab.github.io/2019/09/14/git-comment-specs/)
> - [choerodon_提交命名规约](http://choerodon.io/zh/docs/practice-specification-reference/development/branch-management/)





