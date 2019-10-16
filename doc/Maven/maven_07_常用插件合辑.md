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



## 3.maven-source-plugin

```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.0.1</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```





## 4.maven-shade-plugin

此插件作用于 package 阶段，因此可用此插件打包

> 参考：[十五、使用maven-shade-plugin插件将项目打成可执行的jar包]( https://blog.csdn.net/newbie_907486852/article/details/80921827 )



```xml
 <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>1.4</version>
                <configuration>
                    <createDependencyReducedPom>false</createDependencyReducedPom>
                </configuration>
                <executions>
                    <execution>
                        <!-- 执行package的phase -->
                        <phase>package</phase>
                        <!-- 为这个phase绑定goal -->
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <!-- 过滤掉以下文件，不打包 ：解决包重复引用导致的打包错误-->
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>META-INF/spring.handlers</resource>
                                </transformer>
                                <!-- 打成可执行的jar包 的主方法入口-->
                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>com.yang.MainTest</mainClass>
                                </transformer>

                                <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                    <resource>META-INF/spring.schemas</resource>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```









# 三、其他

## 1.`<resources>`

```xml
<build>
    <!-- 解决target/classes下没有 src/main/resources下的资源文件 -->
    <resources>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.*</include>
        </includes>
      </resource>
      <resource>
        <directory>src/main/resources</directory>
        <includes>
          <include>**/*.*</include>
        </includes>
      </resource>
    </resources>
</build>
```









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

