---
title: macOS安装RabbitMQ

date: 2019-02-17 08:08

updated: 2019-02-17 08:08

tags:
- RabbitMQ

categories: macOS

permalink: rabbitmq-for-mac
---

## 简介

这篇POST将会介绍如何在mac OS安装、运行RabbitMQ。



## 安装

~~~shell
$ brew install rabbitmq
$ export PATH=$PATH:/usr/local/sbin
~~~



确保安装成功

~~~shell
$ brew services list
Name Status User Plist
rabbitmq stopped
~~~



## 启用管理页面

~~~shell
$ rabbitmq-plugins enable rabbitmq_management
~~~



## 添加管理员用户

~~~shell
$ rabbitmqctl add_user yourUsername yourPassword 
$ rabbitmqctl set_user_tags yourUsername administrator
~~~



