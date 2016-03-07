---
layout: post
title: "如何编写和发布博客"
date: 2016-03-07 15:55:49 +0800
comments: true
categories: 
---

### 开启本地监听和http服务器
* rake watch -- 监听源文件变化
* rake preview -- 本地开启http端口，通过访问http://localhost:4000/访问

### 新建文章

* rake new_post['{post name}'] 

注意：文件名命名规则，参考[这里](http://octopress.org/docs/blogging/)


### 发布

* rake generate -- 根据源文件生成目标文件
* rake deploy -- 发布到github
* git push origin source -- 记录修改内容

Done！