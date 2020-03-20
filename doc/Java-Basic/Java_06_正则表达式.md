[toc]





# 前言

## 推荐阅读

> - [正则表达式手册_oschina](https://tool.oschina.net/uploads/apidocs/jquery/regexp.html)
> - 



# 常见用法

## 1.获取网站域名

（1）工具类

```java
public class DomainHelper {

    public static final String REG_DOMAIN_FULL = "(?<=//)(.*?)((?=/)|(?=$))";

    public static String getFullDomain(String url) {
        return getDomain(url, REG_DOMAIN_FULL);
    }

    public static String getTopDomain(String url) {
        String fullDomain = getDomain(url, REG_DOMAIN_FULL);
        String[] split = fullDomain.split("\\.");
        String topDomain = null;
        if(split.length >= 2){
            topDomain = split[split.length-2] + "."+split[split.length-1];
        }else{
            topDomain = fullDomain;
        }
        return topDomain;
    }

    public static String getDomain(String url, String reg) {
        Pattern p = Pattern.compile(reg);
        Matcher m = p.matcher(url);
        String domain = null;
        while (m.find()) {
            domain = m.group();
        }
        return domain;
    }
}

```



（2）测试用例

```java
    @Test
    public void getDoamin() {
        String url1 = "https://tool.oschina.net/uploads/apidocs/jquery/regexp.html";
        String fullDomain1 = DomainHelper.getFullDomain(url1);
        String topDomain1 = DomainHelper.getTopDomain(url1);
        System.out.println("fullDomain1: "+fullDomain1);
        System.out.println("topDomain1: "+topDomain1);


        String url2 = "http://localhost:8080/";
        String fullDomain2 = DomainHelper.getFullDomain(url2);
        String topDomain2 = DomainHelper.getTopDomain(url2);
        System.out.println("fullDomain2: "+fullDomain2);
        System.out.println("topDomai2n: "+topDomain2);
    }

```



输出：

```properties
fullDomain1: tool.oschina.net
topDomain1: oschina.net
fullDomain2: localhost:8080
topDomai2n: localhost:8080
```















