[TOC]

# 前言

插件是在生命周期的各个阶段起作用的







# 常用插件

## 1.maven-compiler-plugin

maven内置的`complier`插件默认编译版本为1.5，若想支持其他的编译版本，则需要自定义其他的编译版本

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>


<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>${maven.compiler.source}</source>
        <target>${maven.compiler.target}</target>
        <encoding>${project.build.sourceEncoding}</encoding>
        <compilerArguments>
            <extdirs>${project.basedir}/lib</extdirs>
        </compilerArguments>
    </configuration>
</plugin>
```



## 2.maven-surefire-plugin

跳过单元测试

```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.5</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
```







# 三、其他

## 1.`tomcat7-maven-plugin`

```xml
          <plugin>
              <groupId>org.apache.tomcat.maven</groupId>
              <artifactId>tomcat7-maven-plugin</artifactId>
              <configuration>
                  <path>/</path>
                  <port>8090</port>
              </configuration>
              <version>2.2</version>
          </plugin>
```

