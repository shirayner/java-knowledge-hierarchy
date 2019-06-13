[TOC]

# 前言





# 一、使用 dockerfile-maven-plugin 自动构建镜像并推送至远程公共仓库



## 1.编写Dockerfile



```dockerfile
# 基于哪个镜像
FROM openjdk:jre-alpine
# 修改时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 将本地文件夹挂载到当前容器
VOLUME /tmp

# 拷贝文件到容器，
ARG JAR_FILE
ADD ${JAR_FILE} app.jar
RUN sh -c 'touch /app.jar'
ENV JAVA_OPTS=""

# 开放8761端口
EXPOSE 8761

# 配置容器启动后执行的命令
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar " ]
```





## 2.dockerfile-maven-plugin配置

（1）在父项目中

```xml
  <pluginManagement>
            <plugins>

                <!-- spring-boot-maven-plugin -->
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <goals>
                                <goal>repackage</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>

                <!-- docker插件 -->
                <plugin>
                    <groupId>com.spotify</groupId>
                    <artifactId>dockerfile-maven-plugin</artifactId>
                    <version>${dockerfile-maven-plugin.version}</version>
                    <executions>
                        <execution>
                            <id>build-image</id>
                            <phase>package</phase>
                            <goals>
                                <goal>build</goal>
                                <goal>tag</goal>
                                <goal>push</goal>
                            </goals>
                        </execution>
                        <execution>
                            <id>tag-image-version</id>
                            <phase>deploy</phase>
                            <goals>
                                <goal>tag</goal>
                                <goal>push</goal>
                            </goals>
                            <configuration>
                                <tag>${project.version}</tag>
                            </configuration>
                        </execution>
                        <execution>
                            <id>tag-image-latest</id>
                            <phase>deploy</phase>
                            <goals>
                                <goal>tag</goal>
                                <goal>push</goal>
                            </goals>
                            <configuration>
                                <tag>latest</tag>
                            </configuration>
                        </execution>
                    </executions>
                    <configuration>
                        <!-- <repository>${docker.registry.name}/${project.artifactId}</repository>-->
                        <repository>${docker-registry-aliyun.name}/${project.artifactId}</repository>
                        <tag>${project.version}</tag>
                        <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
                        <buildArgs>
                            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                        </buildArgs>
                    </configuration>
                    <dependencies>
                        <!-- To make this work on JDK 9+ -->
                        <dependency>
                            <groupId>javax.activation</groupId>
                            <artifactId>javax.activation-api</artifactId>
                            <version>1.2.0</version>
                        </dependency>
                    </dependencies>
                </plugin>
            </plugins>
        </pluginManagement>
```



（2）在子项目中

```xml
 <build>
        <finalName>${project.artifactId}-${project.version}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```



## 3.远程仓库认证配置

注意上面配置了

```xml
                    <configuration>
                        <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
                    </configuration>
```

因此会采用Maven的 Settings.xml中的配置阿里云docker镜像仓库的认证信息

```
    <server>
      <id>registry.cn-shanghai.aliyuncs.com</id>
      <username>shiraydsds@qq.com</username>
      <password>dsdfafdsds</password>
    </server>
```





## 4.自动构建镜像并上传

命令行运行如下命令即可

```
mvn clean package
```

















