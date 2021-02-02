# 基于VuePress和GitHub搭建个人博客站点

[toc]



## 推荐阅读

> - [vuepress + GitHub搭建个人博客笔记（1）](https://segmentfault.com/a/1190000022666197)
> - [使用Vuepress搭建博客](http://www.inode.club/webframe/tool/vuepressBlog.html)
> - [VuePress搭建技术网站与个人博客](https://www.jianshu.com/p/37509da5a020)
> - 





## 一、起步

### 1.安装 vuepress 并初始化项目

```bash
## 1.全局安装 vuepress
npm install -g vuepress 

## 2.创建并进入博客目录
mkdir knowledge-hierarchy
cd knowledge-hierarchy

## 3.初始化项目
npm init -y
```

初始化完成后，会创建一个 `package.json`

```json
{
  "name": "knowledge-hierarchy",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```



### 2.创建文件树结构

创建如下的文件树结构

```less
knowledge-hierarchy
 │  
 ├─package.json
 │  
 └─docs
    ├─.vuepress
    │  │  
    │  ├─public                  存放图片等静态资源
    │  │  
    │  ├─styles
    │  │  └─palette.styl         主题样式->全局
    │  │ 
    │  └─config.js               vuepress配置
    │          
    ├─guide                       
    │   └─README.md
    │ 
    └─README.md
```

> - 参见： [ 目录结构_VuePress](https://www.vuepress.cn/guide/directory-structure.html)
> - `README.md`是必须的，所有的文件路径都是相对于docs目录的，如使用路由`/guide/`来访问 `/guide/README.md`

各文件内容如下：



#### 2.1 config.js

```js
module.exports = {
  title: '游客17846',
  description: 'Just do it!',
  head: [// 会加入<head>中
    ['link', { rel: 'icon', href: '/logo.ico' }],// 指定浏览器Tab图标
    ['link', { rel: 'manifest', href: '/manifest.json' }],//PWA
    ['link', { rel: 'apple-touch-icon', href: '/logo.png' }]// 指定safari浏览器保存书签至桌面图标
  ],
  serviceWorker: true,
  base: '/',// 部署时指定存放的项目的地址
  markdown: {
    lineNumbers: true// 代码块行号显示
  },
  themeConfig: {
    logo: '/logo.png',// 主页显示图标
    nav: [
      { text: '首页', link: '/' },// 首页地址不想指定的话就不用改，默认指向docs下面的README.md
      { text: '博文', link: '/blog/' },// 默认指向blog下的README.md
      { text: 'GitHub', link: 'https://github.com/UAERNAME' }
    ],
    lastUpdated: '上次更新时间'// 页面最下方的最后更新时间戳
  }
};
```



参考：

> - [vuepress基本配置官方文档](https://www.vuepress.cn/guide/basic-config.html#配置文件)
> - [PWA](https://developer.mozilla.org/zh-CN/docs/Web/Manifest)



#### 2.2 palette.styl

```stylus
$accentColor = #87cefa        // 主题色
$textColor = #000             // 文字颜色
$borderColor = #eaecef        // 边框颜色
$codeBgColor = #282c34        // 代码背景颜色
```

这是全局样式文件，可以**自己设置主题色**等，另外在写自己的插件或页面时，**这些样式可以继承使用**



#### 2.3 package.json

在 `package.json`中配置启动命令

> ```json
> "scripts": {
>     "docs:dev": "vuepress dev docs",
>     "docs:build": "vuepress build docs"
>  }
> ```
>
> - 启动项目: `npm run docs:dev`这条命令就等于`vuepress dev docs`
>
> - 打包项目: `npm run build` 这条命令就等于 `vuepress build docs`



### 3.本地运行

```bash
npm run docs:dev
```

VuePress 会在 http://localhost:8080启动一个热重载的开发服务器。







