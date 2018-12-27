# 一、前言







# 二、具体过程

## 1.settings.xml

在maven的 settings.xml中增加如下私服的配置

```xml
  <servers>
     <server>
       <id>releases</id>
       <username>hec-deployer</username>
       <password>123456</password>
     </server>
     <server>
       <id>snapshots</id>
       <username>hec-deployer</username>
       <password>123456</password>
     </server>
     <server>
       <id>3rd</id>
       <username>hec-deployer</username>
       <password>123456</password>
     </server>
   </servers>

   <mirrors>
     <mirror>
       <id>central</id>
       <mirrorOf>*</mirrorOf>
       <url>http://nexus.saas.hand-china.com/content/groups/public</url>
     </mirror>
   </mirrors>

   <profiles>
     <profile>
       <id>central</id>

       <repositories>
         <repository>
           <id>central</id>
           <url>http://central</url>
           <releases>
             <enabled>true</enabled>
           </releases>
           <snapshots>
             <enabled>true</enabled>
           </snapshots>
         </repository>
       </repositories>

       <pluginRepositories>
         <pluginRepository>
           <id>central</id>
           <url>http://central</url>
           <releases>
             <enabled>true</enabled>
           </releases>
           <snapshots>
             <enabled>true</enabled>
           </snapshots>
         </pluginRepository>
       </pluginRepositories>

     </profile>
   </profiles>

   <activeProfiles>
     <activeProfile>
       central
     </activeProfile>
   </activeProfiles>
```





## 2.pom.xml

pom.xml中增加如下配置

```xml
    <distributionManagement>
        <repository>
            <id>releases</id>
            <url>http://nexus.saas.hand-china.com/content/repositories/panda-release</url>
        </repository>

        <snapshotRepository>
            <id>snapshots</id>
            <url>http://nexus.saas.hand-china.com/content/repositories/panda-snapshot</url>
        </snapshotRepository>
    </distributionManagement>

    <!-- 私有仓库 -->
    <repositories>
        <repository>
            <id>RDC thirdparty</id>
            <name>RDC thirdparty Repository</name>
            <url>http://nexus.saas.hand-china.com/content/repositories/thirdparty</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>RDC thirdparty</id>
            <name>RDC thirdparty Repository</name>
            <url>http://nexus.saas.hand-china.com/content/repositories/thirdparty</url>
        </pluginRepository>
    </pluginRepositories>
```

