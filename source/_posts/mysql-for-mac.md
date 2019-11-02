---
title: Mac OS安装MySQL

date: 2019-02-15 09:08

updated: 2019-02-15 09:08

tags:
- MySQL

categories: Mac OS

permalink: mysql-for-mac
---

## 简介

这篇POST将会介绍如何在mac OS安装、运行MySQL。



## 安装

~~~shell
$ brew install mysql
~~~



确保安装成功

~~~shell
$ brew services list
Name Status User Plist
mysql stopped
~~~



## root密码

~~~shell
$ mysql_secure_installation
~~~

初次输入这个命令后，MySQL会通过让你回答问题的方式，对安全性进行配置。**其中包括了配置root的密码**，这些问题分别是

- Would you like to setup VALIDATE PASSWORD component?
  - There are three levels of password validation policy:
- Please set the password for root here.

- Remove anonymous users?
- Disallow root login remotely?
- Remove test database and access to it?
- Reload privilege tables now?

根据需要进行配置即可



## 配置文件的路径

使用Homebrew安装的MySQL，配置文件的路径是

`/usr/local/etc/my.cnf`



## 使用MySQL

当MySQL启动时，就可以使用它了

~~~shell
$ brew services list
Name Status User Plist
mysql started deolin /User/deolin/Library/...
~~~



~~~shell
$ mysql -uroot -pYOUR-ROOT-PASSWORD
~~~

当然，也可以通过Navicat等工具来连接它。

