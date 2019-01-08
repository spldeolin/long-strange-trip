---
title: 利用Vagrant搭建本地开发环境

date: 2018-11-25 07:10:00

updated: 2019-01-08 09:41:00

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
$ vagrant box add your-box-name /opt/the-centos7-virtualbox-file.box
$ vagrant init your-box-name
~~~



解释一下这两条命令

- `vagrant box add <name> <path>`的含义是添加一个**box**。可以通过网上下载的box文件路径，或是 https://vagrantcloud.com/search 搜索到的box名称来添加。这条命令的最终产物可以通过`vagrant box list`来查看。

- `vagrant init <name>`的含义是通过一个已添加的**box**，来创建一个**实例**。这条会在命令行所在的目录创建一个`Vagrantfile`文件，它是这台新初始化的虚拟机的配置文件，`Vagrantfile`所在目录被Deolin称之为**实例目录**。这条命令的最终产物是一个**可运行的Vagrant实例**。



## 使用Xshell连接

使用Xshell连接之前，必须启动**实例**

~~~shell
$ cd your-instance-path
$ vagrant up
~~~



- IP端口：默认情况下是127.0.0.1:2222

- 用户名：默认情况下是vagrant

- SSH验证：Public Key，用户密钥位于

  用户目录下的`.vagrant.d/boxes/your-box-name/0/virtualbox/vagrant_private_key`

- su命令密码：vagrant

- 上述的配置都可以在`Vagarntfile`中修改



## 部署常用环境

参考[这篇POST](https://spldeolin.com/posts/centos-softwares/)



## Vagrant常用命令

- 添加、查看、删除**box**

  ~~~shell
  $ vagrant box add your-box-name box-path
  $ vagrant box list
  $ vagrant box remove your-box-name
  ~~~

- 启动、关闭、重启、查看**实例**

  ~~~shell
  $ vagrant up
  $ vagrant halt
  $ vagrant reload
  $ vagrant status
  ~~~

- 生成、查看、删除**快照**、恢复到**快照**

  ~~~shell
  $ vagrant snapshot save "your-snapshot-name"
  $ vagrant snapshot list
  $ vagrant snapshot delete "your-snapshot-name"
  $ vagrant snapshot restore "your-snapshot-name"
  ~~~



## 配置IP映射

- 修改vagrant配置

~~~
  config.vm.network "private_network", ip: "192.168.2.2"
~~~

- 重启

~~~shell
$ vagrant reload
~~~



## 连接Vagrant中的服务

以连接Vagrant中运行的MySQL为例，说明一下如何连接Vagrant中的服务器

MySQL的root用户在默认情况下无法在外部登录，所以需要创建一个guest用户

~~~
mysql > create user 'guest'@'%' identified by 'guest';
mysql > grant all privileges on *.* to 'guest'@'%';
mysql > flush privileges;
~~~



由于在上一步中，IP已经被映射到了`192.168.2.2`，所以可以直接用Navicat连接

![](/images/vagrant-centos-01.png)



使用`Medis`连接运行在vagrant的Redis时，也是同样的道理。



## 导出实例

现在我们有了一台部署好了的Vagrant虚拟机，可以把它导出成box文件，从而备份开发环境。

~~~shell
$ cd your-instance-path
$ vagrant package --base your-box-name --out output-file-path
~~~



## 导入实例

有时我们需要在别的机器上进行开发，一项项地安装开发环境太慢，直接导入Vagrant实例是个不错的方法。

由于在上一步中已经导出了box文件，所以直接初始化这个box文件即可。



