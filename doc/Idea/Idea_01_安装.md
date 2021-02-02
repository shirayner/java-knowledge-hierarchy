# 前言






# 一、Windows下安装
## 1.下载
> [https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)

## 2.安装
默认安装即可



# 二、Linux下安装

## 1.下载解压

去[官网](http://www.jetbrains.com/)下载Linux版本

下载后解压

```bash
# 创建解压目录
sudo mkdir /usr/local/idea/

# 解压至安装目录
sudo tar -zxvf ideaIU-2019.2.2.tar.gz -C /usr/local/idea/
```



## 2.添加软链接

```bash
sudo ln -s /usr/local/idea/idea-IU-192.6603.28/bin/idea.sh /usr/bin/idea
```



## 3.运行

直接命令行执行如下命令即可

```bash
idea
```













