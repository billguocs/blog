---
title: "利用hugo搭建个人博客"
date: 2019-08-26T18:44:29+08:00
lastmod: 2019-08-28
draft: true
tags: [
    "hugo"
]
categories: [
    "Development"
]
---

[Hugo](https://gohugo.io/)是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。以前个人使用的是Hexo，由于一些原因，迁移到Hugo。

### 快速开始

#### 安装Hugo

1. 二进制安装

   到 Hugo Releases 下载对应的操作系统版本的Hugo二进制文件，配置对应的环境变量

#### 生成站点

#### 部署

个人使用Github Pages + [Travis-CI](https://travis-ci.org/)进行部署

##### 在Github端的操作
  
1. 在个人gitbhu上新建一个repo用于存放博客内容，然后拉出一个`gh-pages`的分支，`gh-pages`用于存放生成的博客静态内容。
2. 申请[Personal access tokens](!https://github.com/settings/tokens/new)并记录下来


##### 在[Travis-Ci](https://travis-ci.org/)的操作
  
1. 利用Github账号登入，获取账号下的repo，点击repo，进入settings, 设置`Environment Variables`,配置之前申请的`GITHUB_TOKEN`