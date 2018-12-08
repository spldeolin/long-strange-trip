---
title: MongoDB 4.0 权限设置

date: 2018-08-22 10:47:00

tags:
- MongoDB

categories: Java

permalink: mongodb-4.0-auth
---



## 简介

这篇POST介绍了最新版本的MongoDB在安装后，创建数据库、创建用户、为用户设置权限。

使用之前，先安装[MongoDB](https://spldeolin.com/posts/centos-softwares/#MongoDB)，并进入Mongo命令行。



## 初次使用

刚安装完毕的Mongdb服务仅允许本地IP访问，且任何操作都不做鉴权，所以可以直接进入命令行，进行初始化工作。



## 创建数据库

~~~shell
> use demo_db
~~~

创建成功的话会显示`switch to db demo_db`



## 为数据库创建用户

~~~shell
> db.createUser({ user: "demo_app_name", pwd: "demo_app_password", roles: [{ role: "read", db: "beginning_mind" }] })
~~~

`demo_app_name`、`demo_app_password`指的是用于认证的用户名和密码

`read`指的是角色，对于一个应用来说，一般需要为它创建一个角色是`readWrite`的用户，让应用可以读取、插入数据。



## 启用安全模式

安全模式下，所有连接必须验证用户名和密码

~~~shell
$ vi /etc/mongod.conf
~~~



- 取消`security`的注释，并在`security`下追加`authorization: enabled`
- 将`net.bindIp`改为`0.0.0.0`，令外部服务也可以访问Mongo



重启MongoDB即可启用

~~~shell
$ service mongod restart
~~~



## 其他

如果你跟Deolin一样，使用`MongoDB Compass`，并在重启`MongoDB`后再次连接报出**command hostInfo requires authentication**错误，那么应该杀掉`MongoDB Compass`并重启该软件