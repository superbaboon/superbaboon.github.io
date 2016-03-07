---
layout: post
title: "常用mac执行命令"
date: 2016-03-07 15:40:12 +0800
comments: true
categories: 
---

## 启动mysql

```
sudo /usr/local/mysql/bin/mysqld_safe
```

## 重新设置root密码

```
1. sudo /usr/local/mysql/bin/mysqld_safe --skip-grant-tables
2. mysql -u root
3. UPDATE mysql.user SET Password=PASSWORD('password') WHERE User='root';
4. FLUSH PRIVILEGES;

```

## 清理rvm

```
1. rvm implode
2. 确保~/.rvm和/usr/local/rvm目录已经被删除
3. 确保~/.bashrc，~/.bash_profile, ~/.zshrc，~/.profile相关的内容已经删除

```