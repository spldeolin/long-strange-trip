---
title: macOS安装Redis

date: 2019-02-15 09:26

updated: 2019-02-15 09:26

tags:
- Redis

categories: macOS

permalink: redis-for-mac
---

## 简介

这篇POST将会介绍如何在mac OS安装、运行Redis。



## 安装

~~~shell
$ brew install redis
~~~



确保安装成功

~~~shell
$ brew services list
Name Status User Plist
redis stopped
~~~



## 配置文件的路径

使用Homebrew安装的Redis，配置文件的路径是

`/usr/local/etc/redis.cnf`



## 使用Redis

当Redis启动时，就可以使用它了

~~~shell
$ brew services list
Name Status User Plist
redis started deolin /User/deolin/Library/...
~~~



~~~shell
$ redis-cli

> auth YOUR-REDIS-AUTH-PASSWORD
OK
> ping
PONG
~~~

当然，也可以通过Redis Desktop Manager等工具来连接它。

