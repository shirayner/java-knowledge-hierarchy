[toc]









# 前言







# 二、常用技巧



## 1.去重

参考：[分享几种 Java8 中通过 Stream 对列表进行去重的方法](https://juejin.im/post/5cd6b719f265da03b2044d56)



```java
            List<User> userList = UserMapper.findByEmailNonNull().stream()
                            .collect(collectingAndThen(toCollection(
                                            () -> new TreeSet<>(Comparator.comparing(User::getEmail))),
                                            ArrayList::new));
```







## 2.排序

```java
taskSetList.stream().sort((o1, o2) -> o1.getId() - o2.getId());

taskSetList.stream().sorted(Comparator.comparing(TaskSet::getId).reversed()).findFirst()
    
securityResourceList.stream().sorted(Comparator.comparing(securityResource -> securityResource.getResourcePath().toLowerCase())).collect(Collectors.toList());

securityResourceList.stream().sorted(Comparator.comparing(SecurityResource::getDatachangeLasttime)).collect(Collectors.toList());
```



