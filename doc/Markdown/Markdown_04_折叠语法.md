[TOC]



# 一、折叠语法

主要使用的是 `html5`的 `details`标签



（1）示例如下：

```xml
<details>
  <summary>折叠文本</summary>
  此处可书写文本
  嗯，是可以书写文本的
</details>


<details>
  <summary>折叠代码块</summary>
  <pre><code> 
     System.out.println("虽然可以折叠代码块");
     System.out.println("但是代码无法高亮");
  </code></pre>
</details>

<details>
  <summary>折叠代码块</summary>
  <pre><blockcode> 
     System.out.println("虽然可以折叠代码块");
     System.out.println("但是代码无法高亮");
  </blockcode></pre>
</details>
```



（2）效果如下：

<details>
  <summary>折叠文本</summary>
  此处可书写文本
  嗯，是可以书写文本的
</details>
<details>
  <summary>折叠代码块</summary>
  <pre><code> 
     System.out.println("虽然可以折叠代码块");
     System.out.println("但是代码无法高亮");
  </code></pre>
</details>





（3）解读

> - `details`：折叠语法标签
> - `summary`：折叠语法展示的摘要
> - `pre`：以原有格式显示元素内的文字是已经格式化的文本
> - `code`：指定代码块





# 参考资料

1. [【MarkDown】使用Html样式和折叠语法](https://www.cnblogs.com/buwuliao/p/9578918.html)
2. [让你的md文档可折叠化展示 #155](https://github.com/iuap-design/blog/issues/155)