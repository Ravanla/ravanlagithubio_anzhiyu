---
title: My New Post
date: 2024-03-27 18:56:39
tags:
---
记录一下用了anzhiyu主题的新网站，基本所有的内容都重新写了，github action也没有设置了

现在记录一下我是如何搭建的anzhiyu主题的

首先要用hexo初始化一个博客框架，这里我试验过直接从github上pull下我在windows上的butterfly主题的播客项目是不行的，hexo无法识别我pull或者clone下来的项目，就算我初始化一个hexo项目了把我windows上的项目覆盖掉hexo也是识别不了的。

所以我是初始化hexo项目，在npm环境都搭建好的情况下，直接安装hexo主题。

## 让github托管项目
在根目录下的_config.yml最后一栏设置
```yml
deploy:
  type: 'git'
  repository: git@github.com:Ravanla/ravanla.github.io.git
  branch: main
```
还有这个url也要设置
```yml
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: https://ravanla.github.io
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks
```

## picgo图床
网上有picgo配置教程，我把我搭建过程中遇到的写在这

![](https://wowmac.oss-cn-shenzhen.aliyuncs.com/account/240327.jpg)

![](https://wowmac.oss-cn-shenzhen.aliyuncs.com/account/240327(1).jpg)

oss配置如下

![](https://wowmac.oss-cn-shenzhen.aliyuncs.com/account/240327(2).jpg)

## 关联域名
ravanla.top

![](https://wowmac.oss-cn-shenzhen.aliyuncs.com/account/240327(3).jpg)

![](https://wowmac.oss-cn-shenzhen.aliyuncs.com/account/240327(4).jpg)

OK