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











