

[TOC]







```
ResourceLoader resourceLoader = new DefaultResourceLoader();
InputStream is = resourceLoader.getResource(fileName).getInputStream();
```





>  java中jar包内的类访问jar包内部的资源文件的路径问题：
>
> http://blog.csdn.net/mm_bit/article/details/50372229
>
> 获取jar包内部的资源文件：
>
> http://blog.csdn.net/luo_jia_wen/article/details/50057191
>
> 【解惑】深入jar包：从jar包中读取资源文件：
>
> http://www.iteye.com/topic/483115
>
>  jar读取资源配置文件，jar包内包外，以及包内读取目录的方法：
>
> http://blog.csdn.net/T1DMzks/article/details/75099029
>
>  java加载jar包下的资源文件过程及原理分析：
>
> http://blog.csdn.net/puhaiyang/article/details/77409203







```java
static URL locateFromClasspath(String resourceName) {
    URL url = null;
    ClassLoader loader = Thread.currentThread().getContextClassLoader();
    if (loader != null) {
        url = loader.getResource(resourceName);
        if (url != null) {
            LOG.debug("Loading configuration from the context classpath (" + resourceName + ")");
        }
    }

    if (url == null) {
        url = ClassLoader.getSystemResource(resourceName);
        if (url != null) {
            LOG.debug("Loading configuration from the system classpath (" + resourceName + ")");
        }
    }

    return url;
}
```







```java
Thread.currentThread().getContextClassLoader().getResource("uncertain.xml");

ClassLoader.getSystemResource("uncertain.xml");

this.getClass().getClassLoader().getResource("gld_responsibility_center.screen").getPath();

GetResource.class.getClassLoader().getResource("gld_responsibility_center.screen").getPath();

this.getServletContext().getRealPath("gld_responsibility_center.screen").isEmpty();

```





