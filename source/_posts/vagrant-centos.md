---
title: 利用Vagrant搭建本地开发环境

date: 2018-11-25 07:10:00

tags:
- Linux

categories: Java

permalink: vagrant-centos
---



## 事前准备

- 下载[Virtualbox](https://www.virtualbox.org/wiki/Downloads)
- 下载[Vagrant](https://www.vagrantup.com/downloads.html)
- 下载[CentOS7 box镜像](https://vagrantcloud.com/centos/boxes/7/versions/1809.01/providers/virtualbox.box)



## 初始化

~~~shell
$ vagrant box add centos7 /opt/the-centos7-virtualbox-file.box
$ vagrant init centos7
~~~



`init`命令执行完毕后，当前目录会产生一个`Vagrantfile`，里面是vagrant虚拟机相关的配置，每次修改vagrant配置指的便是编辑这个文件。

`up`命令执行完毕后，虚拟机正式启动



## 使用Xshell连接

- IP端口：默认情况下是127.0.0.1:2222
- 用户名：默认情况下是vagrant
- SSH验证：Public Key，用户密钥路径为`.vagrant\machines\default\virtualbox\private_key`
- su命令密码：vagrant



## 部署常用环境

参考[这篇POST](https://spldeolin.com/posts/centos-softwares/)



## Vagrant常用命令

- 启动、关闭、重启

  ~~~shell
  vagrant up
  vagrant halt
  vagrant reload
  ~~~

- 查看快照、生成快照、恢复到快照、删除快照

  ~~~shell
  vagrant snapshot list
  vagrant snapshot save "name_for_your_new_snapshot"
  vagrant snapshot go "snapshot_name"
  vagrant snapshot delete "snapshot_name"
  ~~~



## IP映射

- 修改vagrant配置

~~~
  config.vm.network "private_network", ip: "192.168.2.2"
~~~

- 重启

~~~shell
vagrant reload
~~~



## 连接Vagrant MySQL

MySQL的root用户在默认情况下无法在外部登录，所以需要创建一个guest用户

~~~
mysql > create user 'guest'@'%' identified by 'guest';
mysql > grant all privileges on *.* to 'guest'@'%';
mysql > flush privileges;
~~~



由于在上一步中，IP已经被映射到了`192.168.2.2`，所以可以直接用Navicat连接

![](images/vagrant-centos-01.png)



使用`Medis`连接运行在vagrant的Redis时，也是同样的道理。

