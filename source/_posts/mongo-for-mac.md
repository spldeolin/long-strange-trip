---
title: Mac OS安装MongoDB

date: 2019-02-15 10:08

updated: 2019-02-15 10:08

tags:
- MongoDB

categories: Mac OS

permalink: mongo-for-mac
---

## 简介

这篇POST将会介绍如何在mac OS安装、运行MongoDB。



## 安装

~~~shell
$ brew install mongodb
~~~



确保安装成功

~~~shell
$ brew services list
Name Status User Plist
mongodb stopped
~~~



## 配置文件的路径

使用Homebrew安装的Mongo，配置文件的路径是

`/usr/local/etc/mongod.cnf`



## 使用MongoDB

~~~shell
$ mongo

> db.version()
4.0.3
~~~



当然，也可以通过Navicat等工具来连接它。

