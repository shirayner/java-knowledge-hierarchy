[TOC]



# 前言

在上一节  [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md)  中，我们了解了 Spring 声明式缓存抽象，知道了：

> - 我们使用Spring声明式缓存，使用的是其抽象，无关底层实现。也就是说，不管你底层使用的是什么缓存技术，对Spring声明式缓存的使用是一致的
> - 若想要采用不同的缓存技术，只需要替换掉底层注册的 CacheManager 即可



那么，这一节我们将采用 EhCache 作为缓存技术，来实现一个Spring声明式缓存的实例，这显然就很简单了，步骤如下：

> - 引入 EhCache 依赖
> - 开启缓存支持
> - 配置ehcache（ehcache.xml）
> - 使用声明式缓存

其他步骤同上一步： [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md) 



# 一、SpringBoot 整合 EhCache

## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringBoot_01_入门示例](./SpringBoot_01_入门示例.md)

```properties
group = 'com.ray.study'
artifact ='spring-boot-07-cache-ehcache'
```



## 2.引入依赖

### 2.1 继承父工程依赖

在父工程`spring-boot-seeds` 的 `settings.gradle`加入子工程

```properties
rootProject.name = 'spring-boot-seeds'
include 'spring-boot-01-helloworld'
include 'spring-boot-02-restful-test'
include 'spring-boot-03-thymeleaf'
include 'spring-boot-04-swagger2'
include 'spring-boot-05-jpa'
include 'spring-boot-05-mybatis'
include 'spring-boot-05-tk-mybatis'
include 'spring-boot-06-nosql-redis'
include 'spring-boot-06-nosql-mongodb'
include 'spring-boot-07-cache-concurrentmap'
include 'spring-boot-07-cache-ehcache'
```



这样，子工程`spring-boot-07-cache-ehcache`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
		implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```



### 2.2 引入`ehcache` `依赖

将子模块`spring-boot-07-cache-ehcache` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    // spring-boot-starter-cache  和 ehcache
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'net.sf.ehcache:ehcache'

    // mysql 驱动和 jpa
    implementation 'mysql:mysql-connector-java'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}

```



## 3.数据库准备

同上一节： [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md) 



## 4.修改配置

### 4.1 修改`application.yml`

同上一节： [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md) 

### 4.2 CacheConfiguration

开启缓存支持

同上一节： [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md) 

### 4.3 配置ehcache.xml

（1）默认自动加载resources目录下的ehcache.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="ehcache.xsd">
    <!--timeToIdleSeconds 当缓存闲置n秒后销毁 -->
    <!--timeToLiveSeconds 当缓存存活n秒后销毁 -->
    <!-- 缓存配置
        name:缓存名称。
        maxElementsInMemory：缓存最大个数。
        eternal:对象是否永久有效，一但设置了，timeout将不起作用。
        timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
        timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
        overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。 diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
        maxElementsOnDisk：硬盘最大缓存个数。
        diskPersistent：是否缓存虚拟机重启期数据 Whether the disk
        store persists between restarts of the Virtual Machine. The default value
        is false.
        diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。  memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是
LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
        clearOnFlush：内存数量最大时是否清除。 -->
    
    
    <!-- 磁盘缓存位置 -->
    <diskStore path="java.io.tmpdir" />
    <!-- 默认缓存 -->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">

        <persistence strategy="localTempSwap" />
    </defaultCache>

    <!-- 测试 -->
    <cache name="user"     
           eternal="false"
           timeToIdleSeconds="2400"
           timeToLiveSeconds="2400"
           maxEntriesLocalHeap="10000"
           maxEntriesLocalDisk="10000000"
           diskExpiryThreadIntervalSeconds="120"
           overflowToDisk="false"
           memoryStoreEvictionPolicy="LRU">
    </cache>
</ehcache>
```



 这里配置了一个 cacheName为 user 的cache，实际使用时指定的缓存名称必须是在此配置文件中存在的，否则会抛找不到cache的异常

```java
@CachePut(value = "user", key = "#user.name")  // 这里的cacheName（即user）必须在ehcache.xml中存在
public User save(User user) {
    User u = userRepository.save(user);
    log.info("新增：缓存用户，用户id为:{}", u.getId());
    return u;
}
```





（2）对于EhCache的配置文件也可以通过`application.yml`文件中使用`spring.cache.ehcache.config`属性来指定，比如：

```yml
spring:
  cache:
    ehcache:
      config: classpath:ehcache.xml
```



（3）**ehcache.xml配置文件详解**

> - `diskStore`：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。
> - `defaultCache`：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
> - `name`:缓存名称。
> - `maxElementsInMemory`:缓存最大数目
> - `maxElementsOnDisk`：硬盘最大缓存个数。
> - `eternal`:对象是否永久有效，一但设置了，timeout将不起作用。
> - `overflowToDisk`:是否保存到磁盘，当系统宕机时
> - `timeToIdleSeconds`:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
> - `timeToLiveSeconds`:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
> - `diskPersistent`：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
> - `diskSpoolBufferSizeMB`：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
> - `diskExpiryThreadIntervalSeconds`：磁盘失效线程运行时间间隔，默认是120秒。
> - `memoryStoreEvictionPolicy`：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
> - `clearOnFlush`：内存数量最大时是否清除。
>
> 

## 5.业务实现

同上一节： [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md) 

## 6.单元测试

同上一节： [SpringBoot_07_Cache_01_快速入门](./SpringBoot_07_Cache_01_快速入门.md) 





# 参考资料

1. [梁桂钊__Spring Boot 揭秘与实战（二） 数据缓存篇 - EhCache](http://blog.720ui.com/2017/springboot_02_data_cache_ehcache/)
2. [SpringBoot学习－（十八）SpringBoot整合EhCache](https://blog.csdn.net/qq_28988969/article/details/78210873)

