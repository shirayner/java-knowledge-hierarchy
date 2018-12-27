# 一、前言
> 此系列随笔是针对[《深入理解Java虚拟机:JVM高级特性与最佳实践 第2版》](https://www.amazon.cn/dp/B00D2ID4PK/ref=sr_1_1?s=books&ie=UTF8&qid=1538275437&sr=1-1&keywords=%E6%B7%B1%E5%85%A5java%E8%99%9A%E6%8B%9F%E6%9C%BA)的总结


上一节，我们搭建好了java的开发环境，这一节，我们来看Java的技术体系


# 二、Java技术体系
## 1.按功能划分

如 果 仅 从传 统 意 义 上 来 看， Sun 官 方 所 定 义 的 Java 技 术 体 系 包 括 以 下 几 个 组 成 部 分： 
> - Java程序设计语言
> - Java API，包括 Java API类库 和 来自商业机构以及开源社区的第三方类库
> - Class 文件格式
>
> - 各种硬件平台上的Java 虚 拟 机 




### 1.1 名词解释
| 简称    | 全称                           | 含义                           |
| ------- | ------------------------------ | ------------------------------ |
| Java SE | Java Platform Standard Edition | java 平台标准版                |
| JDK     | Java SE Development Kit        | Java 语言的软件开发工具包(SDK) |
| JRE     | Java Runtime Environment       | Java运行时环境                 |
| JVM     | Java Virtual Machine           | Java虚拟机                     |

### 1.2 JDK
> 全称：Java SE Development Kit —— Java 语言的软件开发工具包(SDK)
>
> 我们可以把 `Java程序设计语言`、`Java API类库`、`Java虚拟机`这三部分统称为 JDK，JDK是用于支持Java程序开发的最小环境。

JDK的组成结构如下图（来自 [JDK8官方文档](https://docs.oracle.com/javase/8/docs/) 首页）
![在这里插入图片描述](images/20180930004731422)


### 1.3 JRE
> 全称：Java Runtime Environment —— Java运行时环境
>
> 如上图，我们可以把Java API类库中的`Java SE API子集`和`Java虚拟机`这两部分统称为 JRE（ Java Runtime Environment），JRE是支持Java程序运行的标准环境。

### 1.4 JVM
> 全称：Java Virtual Machine —— Java虚拟机
>
> JVM是一种用于计算设备的规范，它是一个虚构出来的计算机，是通过在实际的计算机上仿真模拟各种计算机功能来实现的。

### 1.5 jdk、jre、jvm的关系
三者的关系为：
>JDK包含了JRE
>JRE包含了JVM

![在这里插入图片描述](images/20180930003346781)

## 2.按业务领域划分
如 果 按 照 技 术 所 服 务 的 领 域 来 划 分， 或 者 说 按 照 Java 技 术 关 注 的 重 点 业 务 领 域 来 划 分， Java 技 术 体 系 可 以 分 为 4 个 平 台， 分 别 为： 
> - Java Card
> >支 持 一 些 Java 小 程 序（ Applets） 运 行 在` 小 内 存 设 备`（ 如 智 能 卡） 上 的 平 台。
> - Java ME（ Micro Edition）
> > 支 持 Java 程 序 运 行 在 `移 动 终 端`（ 手 机、 PDA） 上 的 平 台， 对 Java API 有 所 精 简， 并 加 入 了 针 对 移 动 终 端 的 支 持， 这 个 版 本 以 前 称 为 J2ME。 
> - Java SE（ Standard Edition）
> > 支 持 面 向 `桌 面 级 应 用`（ 如 Windows 下 的 应 用 程 序） 的 Java 平 台， 提 供 了 完 整 的 Java 核 心 API， 这 个 版 本 以 前 称 为 J2SE。
> > -  Java EE（ Enterprise Edition）
> >   支 持 使 用 多 层 架 构 的 `企 业 应 用`（ 如 ERP、 CRM 应 用） 的 Java 平 台， 除 了 提 供 Java SE API 外， 还 对 其 做 了 大 量 的 扩 充[ 3] 并 提 供 了 相 关 的 部 署 支 持， 这 个 版 本 以 前 称 为 J2EE。

# 三、Java的优点
Java能获得如此广泛的认可，除了它拥有一门`结构严谨、面向对象`的编程语言之外，还有许多不可忽视的优点：
> 1. 跨平台
> >它摆脱了硬件平台的束缚，实现了“一次编写、到处运行”
> 2. 相对安全的内存管理和访问机制
> > 它提供了一个相对安全的内存管理和访问机制，避免了绝大部分的内存泄露和指针越界问题
> 3. 热点代码检测和运行时编译及优化
> >它实现了热点代码检测和运行时编译及优化，这使得Java应用能随着运行时间的增加而获得更高的性能
> 4. 完整的应用程序接口和第三方类库
>
> >它有一套完整的应用程序接口，还有无数来自商业机构和开源社区的第三方类库来帮助它实现各种各样的功能



# 四、参考资料
1. [Java Platform Standard Edition 8 Documentation](https://docs.oracle.com/javase/8/docs/)
2. [五月的仓颉——Java虚拟机1：什么是Java](http://www.cnblogs.com/xrq730/p/4826691.html)
3. [弄懂JDK、JRE和JVM到底是什么](https://www.cnblogs.com/mambahyw/p/7978832.html)