[toc]









# 前言







# 二、实例



## 1.筛选

### 1.1 去重

参考：[分享几种 Java8 中通过 Stream 对列表进行去重的方法](https://juejin.im/post/5cd6b719f265da03b2044d56)



```java
            List<User> userList = UserMapper.findByEmailNonNull().stream()
                            .collect(collectingAndThen(toCollection(
                                            () -> new TreeSet<>(Comparator.comparing(User::getEmail))),
                                            ArrayList::new));
```



（2）根据Filed去重

```java
public class DistinctByKey {

    public static <T> Predicate<T> distinctByKey(Function<? super T,Object> keyExtractor) {
        Map<Object,Boolean> seen = new ConcurrentHashMap<>();
        return t -> seen.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;
    }

}
```



用法：

```java
 List<SecurityUserLang> userLangs  =   userLangList.stream()
                        .filter(distinctByKey(userlang-> userlang.getOriginLocale() +":"+ userlang.getTargetLocale()))
                        .map(securityUserLang -> {
                    securityUserLang.setUserId(userId);
                    return securityUserLang;
                }).collect(Collectors.toList());
```







### 1.2 取最大值

```
  Date maxReportDate = qaBagReportList.stream().map(QaBagReport::getReportDate).max(Comparator.comparing(java.util.Date::getTime)).get();
```





## 2.排序

```java
taskSetList.stream().sort((o1, o2) -> o1.getId() - o2.getId());

taskSetList.stream().sorted(Comparator.comparing(TaskSet::getId).reversed()).findFirst()
    
securityResourceList.stream().sorted(Comparator.comparing(securityResource -> securityResource.getResourcePath().toLowerCase())).collect(Collectors.toList());

securityResourceList.stream().sorted(Comparator.comparing(SecurityResource::getDatachangeLasttime)).collect(Collectors.toList());
```



（2）自定义排序

```
public class Sort {
    
    public List<User> sort(List<User> users){
        return users.stream().sorted((u1, u2) -> u1.getCreatedTime().compareTo(u2.getCreatedTime())).collect(Collectors.toList());
    }

}
```





## 3.映射

### 3.1 map

将流中每一个元素从一种类型，映射为另一种类型

```java
List<Long> resuourceIds = securityRoleResourceList.stream().map(SecurityRoleResource::getResourceId).collect(Collectors.toList())
```





### 3.2 flatMap

将流扁平化

> - [java8中stream的map和flatmap的理解](https://www.cnblogs.com/lijingran/p/8727507.html)
> - [Java8:如何使用flatMap()](https://www.jianshu.com/p/8d80dcb4e7e0)



```java
            List<Long> bagTaskInfoIdsInTranslatingAndProjectContainTargetLocale = projectRelationList.stream().filter(projectRelation -> {
                // if locale in targetLocales then return true
                List<String> relationTargetLocales = Arrays.asList(projectRelation.getTargetLocales().split(","));
                Optional<String> inTargetLocaleOptional = relationTargetLocales.stream().filter(targetLocaleList::contains).findAny();
                if (inTargetLocaleOptional.isPresent()) {
                    return true;
                }
                return false;
            }).map(projectRelation -> relationIdToBagTaskInfosMap.get(projectRelation.getId()).stream().map(ProjectRelationBagtaskInfo::getBagTaskInfoId).collect(Collectors.toList())).flatMap(Collection::stream).collect(Collectors.toList());
            return bagTaskInfoIdsInTranslatingAndProjectContainTargetLocale;
```





## 4.分组

> 分组归一化操作都可用 reducing 

### 4.1分组求最值

> [java stream 处理分组后取每组最大](https://blog.csdn.net/kingmax54212008/article/details/102827306)

```
Map<String, HitRuleConfig> configMap = configList.parallelStream().collect(
 　　　　　　　　　　　　　　Collectors.groupingBy(HitRuleConfig::getAppId, // 先根据appId分组
　　　　　　　　　　　　　　 Collectors.collectingAndThen(
　　　　　　　　　　　　　　 Collectors.reducing(( c1,  c2) -> c1.getVersionSort() > c2.getVersionSort() ? c1 : c2), Optional::get)));
```



### 4.2 分组求和





