# 一、前言

除了坐标、依赖以及仓库之外，Maven另外两个核心概念是生命周期和插件。



**生命周期是抽象，插件是其具体实现。**



# 二、生命周期

Maven的生命周期就是对所有的构建过程进行抽象和统一，包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有构建步骤。

Maven的生命周期是抽象的，这意味着生命周期本身不做任何实际的工作，在Maven的设计中，实际的任务（如编译源代码）都交由插件来完成。这种思想与设计模式中的模板方法（TemplateMethod）非常相似。

每个构建步骤都可以绑定一个或者多个插件行为，而且Maven为大多数构建步骤编写并绑定了默认插件。例如，针对编译的插件有maven-compiler-plugin，针对测试的插件有maven-surefire-plugin等。



## 1.三套生命周期

maven有三套生命周期：clean、 default、site

三套生命周期相互独立，每个生命周期包含一些阶段（phase），这些阶段是有顺序的，并且后面的阶段依赖于前面的阶段，



### 1.1 clean 

清理项目

它包含三个阶段：

1）pre-clean执行一些清理前需要完成的工作。

2）clean清理上一次构建生成的文件。

3）post-clean执行一些清理后需要完成的工作。





### 1.2 default

定义了项目构建时所需要执行的所有步骤

这里只列出比较重要的阶段：

- process-sources：处理项目主资源文件。

    > 一般来说，是对src/main/resources目录的内容进行变量替换等工作后，复制到项目输出的主classpath目录中。

- compile：编译项目的主源码。

    > 一般来说，是编译src/main/java目录下的Java文件至项目输出的主classpath目录中。

- process-test-sources：处理项目测试资源文件。

    > 一般来说，是对src/test/resources目录的内容进行变量替换等工作后，复制到项目输出的测试classpath目录中。

- test-compile：编译项目的测试代码。

    > 一般来说，是编译src/test/java目录下的Java文件至项目输出的测试classpath目录中

- test：使用单元测试框架运行测试，测试代码不会被打包或部署。

- package：接受编译好的代码，打包成可发布的格式，如JAR。

- install：将jar包安装到Maven本地仓库，供本地其他Maven项目使用。

- deploy：将最终的包复制到远程仓库，供其他开发人员和Maven项目使用。





### 1.3 site

建立和发布项目站点，Maven能够基于POM所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息。

该生命周期包含如下阶段：

- pre-site：执行一些在生成项目站点之前需要完成的工作。
- site：生成项目站点文档。
- post-site：执行一些在生成项目站点之后需要完成的工作。
- site-deploy：将生成的项目站点发布到服务器上。





## 2.常用命令行命令

- mvn clean install  :  清理项目，然后安装jar包到maven本地仓库
- mvn test : 执行测试
- mvn clean deploy ： 清理项目，然后发布jar包到远程仓库





# 三、插件

## 1.插件目标

Maven的核心仅仅定义了抽象的生命周期，具体的任务是交由插件完成的，插件以独立的构件形式存在。

一个插件可以完成多个任务，而这每一个任务就是一个插件目标。

> 例如，maven-dependency-plugin 有许多插件目标：dependency:analyze、dependency:tree  和 dependency:list。
>
> **冒号前面是插件前缀，冒号后面是插件目标。**



## 2.插件绑定

Maven生命周期的阶段与插件的目标相互绑定，以完成某个具体的构建任务。



### 2.1 内置绑定

为了能让用户几乎不用任何配置就能构建Maven项目，Maven在核心为一些主要的生命周期阶段绑定了很多插件的目标，当用户通过命令行调用生命周期阶段的时候，对应的插件目标就会执行相应的任务。



### 2.2 自定义绑定

Maven支持将某个插件目标绑定到生命周期的某个阶段上



## 3.插件配置

### 3.1 命令行参数

可通过命令行参数来配置插件，如：

```bash
mvn clean install -Dmaven.test.skip=true
```

参数-D是Java自带的，其功能是通过命令行设置一个Java系统属性





### 3.2 POM中插件全局配置

有些参数的值从项目创建到项目发布都不会改变，或者说很少改变，对于这种情况，在POM文件中一次性配置就显然比重复在命令行输入要方便。

```xml
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.6</source>
          <target>1.6</target>
          <encoding>UTF-8</encoding>
          <compilerArguments>
            <extdirs>${project.basedir}/lib</extdirs>
          </compilerArguments>
        </configuration>
      </plugin>
```



- 通过插件 `<configuration>` 节点可进行插件的全局配置。

- 全局配置的意思是在插件对应的生命周期的所有阶段都采用此配置。



###  3.3 POM中插件任务配置

除了为插件配置全局的参数，用户还可以为某个插件任务配置特定的参数。

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-antrun-plugin</artifactId>
      <version>1.3</version>
      <executions>
        <execution>
          <id>ant-validate</id>
          <phase>validate</phase>
          <goals>
            <goal>run</goal>
          </goals>
          <configuraction>
            <tasks>
              <echo>I'm bound to validate phase.</echo>
            </tasks>
          </configuraction>
        </execution>
        <execution>
          <id>ant-verify</id>
          <phase>verify</phase>
          <goals>
            <goal>run</goal>
          </goals>
          <configuraction>
            <tasks>
              <echo>I'm bound to verify phase.</echo>
            </tasks>
          </configuraction>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```



- execution：执行任务
- id ： 执行任务的id
- phase : 生命周期的特定阶段
- goal ：插件目标
- configuraction ：插件任务的配置



在上述配置中，配置了两个任务：ant-validate 和 ant-verify 。

ant-validate 将插件 maven-antrun-plugin 的 run 目标绑定到maven default生命周期的 validate阶段上，同时配置了一个echo 任务，向命令行输出一段文字。



## 4.获取插件

当需要插件时，可以访问如下网址获取

> [https://maven.apache.org/plugins/](https://maven.apache.org/plugins/)



## 5.插件解析机制

Maven会区别对待依赖的远程仓库与插件的远程仓库：

> - 当Maven需要的依赖在本地仓库不存在时，它会去所配置的远程仓库查找
>
> - 可是当Maven需要的插件在本地仓库不存在时，它就不会去这些远程仓库查找。



不同于repositories及其repository子元素，插件的远程仓库使用pluginRepositories和pluginRepository配置。

默认的中央插件仓库：

> 文件路径：apache-maven-3.5.4/lib/maven-model-builder-3.5.4.jar!/org/apache/maven/model/pom-4.0.0.xml



```xml
  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>
```



远程插件仓库的节点含义与远程仓库的节点含义相同。







