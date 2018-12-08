---
title: 利用CentOS虚拟机搭建本地环境

date: 2018-09-05 09:34:00

tags:
- CentOS

categories: Linux

permalink: vm-centos
---



## 简介

如果使用Windows操作系统开发Java的话，往往需要在Windows搭建很多环境，如各种数据库，各种消息队列等。这些环境，往往需要手动开启，比较麻烦，而如果作成服务的话，又会在不开发的时候影响系统性能。而且，有些环境对Windows支持不太好，比如Redis等。



这篇POST将会介绍利用`VMware Workstation Pro`创建一个`CentOS`虚拟机，在虚拟机中安装环境，让本地项目通过内网IP的形式连接到各种中间件。



## 安装CentOS

1. 下载镜像

    ```http
    http://mirror.nsc.liu.se/centos-store/7.4.1708/isos/x86_64/
    ```

    这里选择CentOS 7，使用DVD版本即可。


2. 创建虚拟机

   由于是本地环境，配置选择1C1G即可

   设备中需要CD/DVD驱动，并指向1. 中下载的iso镜像文件。


3. 安装操作系统

   创建完毕后，启动虚拟机，在虚拟机中按流程安装CentOS系统即可，注意中间有一个步骤需要选择时区



## 设置固定IP

参考这篇文章

~~~http
https://www.linuxidc.com/Linux/2017-12/149910.htm
~~~



## 搭建环境

参考这篇POST

~~~http
https://spldeolin.com/posts/centos-softwares/
~~~



其中，这个第一次使用`wget`命令会报错，需要专门安装一下

~~~shell
$ yum install wget
~~~

