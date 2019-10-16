[TOC]







# 前言



# 一、本地安装 Elasticsearch

参考 [Elasticsearch_01_安装和配置](../Elasticsearch/Elasticsearch_01_安装和配置.md) 进行安装

> 安装时需要注意 springboot 版本要与Elasticsearch版本保持一致，本文使用的 springboot 版本为 springboot-2.1.4.REALESE ，内部依赖的elasticsearch客户端版本为 6.4.3 , 因此需要安装 6.4.3 版本的Elasticsearch



然后在  elasticsearch.yml  配置文件中配置如下参数：

```yml
# 配置集群名称
cluster.name: cluster-1
# 配置节点名称
node.name: node-1
# 指定节点为master节点
node.master: true
# 指定节点需要持久化数据
node.data: true
network.host: 0.0.0.0
# 开启 cors 保护
http.cors.enabled: true 
# 设置允许跨域
http.cors.allow-origin: "*"

```







# 二、SpringBoot 整合Elasticsearch

## 1.创建子模块

这里我们创建一个子模块，创建步骤同 [SpringBoot_01_入门示例](./SpringBoot_01_入门示例.md)

```properties
group = 'com.ray.study'
artifact ='spring-boot-12-elasticsearch'
```



## 2.引入依赖

### 2.1 继承父工程依赖

在父工程`spring-boot-seeds` 的 `settings.gradle`加入子工程

```properties
rootProject.name = 'spring-boot-seeds'
include 'spring-boot-01-helloworld'
include 'spring-boot-02-restful-test'
include 'spring-boot-03-thymeleaf'
include 'spring-boot-03-freemarker'
include 'spring-boot-04-swagger2'
include 'spring-boot-05-jpa'
include 'spring-boot-05-mybatis'
include 'spring-boot-05-tk-mybatis'
include 'spring-boot-06-nosql-redis'
include 'spring-boot-06-nosql-mongodb'
include 'spring-boot-07-cache-concurrentmap'
include 'spring-boot-07-cache-ehcache'
include 'spring-boot-07-cache-caffeine'
include 'spring-boot-07-cache-redis'
include 'spring-boot-08-messaging-rabbitmq'
include 'spring-boot-09-other-sendmail'
include 'spring-boot-10-config-readconfigproperty'
include 'spring-boot-11-spring-security-standard'
include 'spring-boot-12-elasticsearch'
```



这样，子工程`spring-boot-12-elasticsearch`就会自动继承父工程中`subprojects` `函数里声明的依赖，主要包含如下依赖：

```groovy
		implementation 'org.springframework.boot:spring-boot-starter-web'
        testImplementation 'org.springframework.boot:spring-boot-starter-test'

        compileOnly 'org.projectlombok:lombok'
        annotationProcessor 'org.projectlombok:lombok'
```



### 2.2 引入`jpa`依赖

将子模块`spring-boot-05-jpa` 的`build.gradle`修改为如下内容：

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'
}

```



## 3.修改`application.yml`



```yml
server:
  port: 8089

spring:
  data:
    elasticsearch:
      cluster-name: cluster-1
      cluster-nodes: 127.0.0.1:9300


```







## 4.业务实现

### 4.1 entity

（1）BaseDO

```java
package com.ray.study.springboot12elasticsearch.entity;

import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.io.Serializable;

/**
 * UserES
 *
 * @author ray
 * @date 2019/10/15
 */
@Data
@Document(indexName = "blog", type = "blog")
public class BlogES implements Serializable {

    @Id
    private Long id;

    //@Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String author;

    private String title;

    private String summary;

    private String content;


}

```



（2）User

```java
package com.ray.study.springboot05jpa.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.Table;


/**
 * 用户模型
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "user")
public class User extends BaseDO{

	/**
	 * 用户名
	 */
	private String name;

	/**
	 * 年龄
	 */
	private Integer age;


}


```



### 4.2 repository

- UserRepository

```java
package com.ray.study.springboot12elasticsearch.repository;

import com.ray.study.springboot12elasticsearch.entity.BlogES;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * UserESRepository
 *
 * @author ray
 * @date 2019/10/15
 */
@Repository
public interface BlogESRepository extends ElasticsearchRepository<BlogES, Long> {

    List<BlogES> findByTitle(String title);

}

```



### 4.3 controller

```java
package com.ray.study.springboot12elasticsearch.controller;

import com.ray.study.springboot12elasticsearch.entity.BlogES;
import com.ray.study.springboot12elasticsearch.repository.BlogESRepository;
import org.elasticsearch.index.query.QueryBuilders;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.elasticsearch.core.query.NativeSearchQueryBuilder;
import org.springframework.web.bind.annotation.*;

import javax.validation.constraints.Max;
import java.util.ArrayList;
import java.util.List;

/**
 * UserController
 *
 * @author ray
 * @date 2019/10/16
 */
@RestController
@RequestMapping("/blog")
public class BlogController {

    @Autowired
    private BlogESRepository blogESRepository;

    @GetMapping("")
    public List list() {
        Iterable<BlogES> blogESList = blogESRepository.findAll();
        List blogList= new ArrayList();
        blogESList.forEach(blogES -> {
            blogList.add(blogES);
        });
        return blogList;
    }

    @GetMapping("/search")
    public Page search(@RequestParam("keyword") String keyword) {
        // 构建查询条件
        NativeSearchQueryBuilder queryBuilder = new NativeSearchQueryBuilder();
        // 添加基本分词查询
        //queryBuilder.withQuery(QueryBuilders.matchQuery("author", keyword));
        queryBuilder.withQuery(QueryBuilders.matchPhraseQuery("author", keyword));
        // 搜索，获取结果
        Page<BlogES> blogESPage = blogESRepository.search(queryBuilder.build());
        return blogESPage;
    }

    @PostMapping("/add")
    public BlogES create(@RequestBody BlogES blogES) {
        return blogESRepository.save(blogES);
    }

}

```











# 参考资料

1. [elasticsearch入门 springboot2集成elasticsearch 实现全文搜索,图文讲解带源码](https://juejin.im/post/5c9700956fb9a0710c703e06)
2. [Elasticsearch实战篇——Spring Boot整合ElasticSearch](https://segmentfault.com/a/1190000018625101)
3. 











