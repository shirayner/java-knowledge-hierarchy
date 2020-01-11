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







