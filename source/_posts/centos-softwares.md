---
title: CentOS常用环境、软件

date: 2018-04-16 17:01:00

updated: 2019-01-12 21:55:00

tags:
- 总结
- MySQL
- Redis
- MongoDB
- Nginx
- NodeJS
- Maven
- Git
- Zookeeper
- RabbitMQ
- Elasticsearch
- CentOS
- Python

categories: Linux

permalink: centos-softwares
---



## 简介

这篇POST介绍了如何为一台全新的CentOS服务器安装常用的环境和软件。用于演示的服务器是一个全新的阿里云ECS，操作系统是CentOS 7.4。



## yum

~~~shell
$ sudo yum update -y
~~~



## OpenJDK

选用JDK8最新版

1. 安装

   ~~~shell
   $ sudo yum install -y java-1.8.0-openjdk-devel
   ~~~

2. 安装完成

   ~~~shell
   $ java -version
   ~~~




## Python 3

选用yum源最新的3.6版本

1. 安装

   ~~~shell
   $ sudo yum install -y centos-release-scl
   $ sudo yum install -y rh-python36
   ~~~

   ~~~shell
   $ scl enable rh-python36 bash
   ~~~

2.  安装完成

   ~~~shell
   $ python -V
   ~~~



## MySQL

选用8.x版本

1. 下载

   ~~~shell
   $ wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
   ~~~

2. 安装

   ~~~shell
   $ sudo rpm -ivh mysql80-community-release-el7-1.noarch.rpm
   $ sudo yum install -y mysql-server
   ~~~

3. 启动

   ~~~shell
   $ sudo service mysqld start
   ~~~

4. 修改root密码

   ~~~shell
   $ sudo grep 'temporary password' /var/log/mysqld.log
   $ mysql -uroot -pTheInitPassword
   ~~~

   ~~~mysql
   mysql > ALTER USER 'root'@'localhost' IDENTIFIED BY 'root_r0oT';
   mysql > exit
   ~~~

   ~~~shell
   $ mysql -uroot -proot_r0oT
   ~~~

   > MySQL在初次安装时会生成一个默认的随机密码，它记录在`/var/log/mysqld.log`中

   > `root_r0oT`可以替换为其他内容，它将作为root的新密码。由于MySQL5.6之后自带了validate_password插件，一般不允许设置过于简单的密码。

5. 创建用户

   新建用户

   ~~~mysql
   mysql > CREATE USER 'guest'@'%' IDENTIFIED BY 'guest_gUe5t';
   ~~~

   > `guest`和`guest_gUe5t`可以替换为其他内容，它们将分布作为新用户的用户名和密码

   授权

   ~~~mysql
   mysql > GRANT INSERT, UPDATE, SELECT ON demo_app.* TO 'guest'@'%';
   ~~~

   > `INSERT, UPDATE, SELECT`代表用户只有插入数据，更新数据，查询数据的权限
   >
   > 如果希望该用户有全部权限，可以替换为`ALL`

   > `demo_app.*`代表用户只有`demo_app`数据库所有表的权限（上面指定的权限）

   刷新授权

   ~~~mysql
   mysql > flush privileges;
   ~~~

6. 字符集

   修改CentOS`/etc/my.cnf`文件

   - 在 `[mysqld]`下追加`character_set_server=utf8`
   - 追加`[mysql]`，并在其下追加`default-character-set=utf8`

7. Communications link failure问题

   这个问题一般是在app运行期间发生的，如果app长时间不访问MySQL，下一次访问MySQL时可能会报错。

   解决方式是在`/etc/my.cnf`的`[mysqld]`下追加`wait_timeout=31536000`和`interactive_timeout=31536000`，然后重启MySQL。



## Redis

1. 引入REMI源

   ~~~shell
   $ sudo  yum install -y http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
   ~~~

2. 下载、安装

   ~~~shell
   $ sudo yum --enablerepo=remi install -y redis
   ~~~

3. 配置

   ~~~shell
   $ sudo vi /etc/redis.conf
   ~~~

   - 使Redis能够被远程访问

     ~~~shell
     # bind 127.0.0.1
     ~~~

     > 注释掉

   - 使Redis以进程形式启动

     ~~~
     daemonize yes
     ~~~

   - Redis密码

     ~~~shell
     requirepass yourPasswordHere
     ~~~

4. 启动

   ~~~shell
   $ sudo service redis start
   ~~~

  5. 设置Redis开机自启动

     ~~~shell
     $ sudo chkconfig redis on
     ~~~


## MongoDB

1. 下载、安装

   ~~~shell
   $ sudo vi /etc/yum.repos.d/mongodb.repo
   ~~~

   ~~~
   [MongoDB]
   name=MongoDB Repository
   baseurl=http://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
   gpgcheck=0
   enabled=1
   ~~~

   ~~~shell
   $ sudo yum install -y mongodb-org
   ~~~

   > 这里选择4.0版本

2. 启动

   ~~~shell
   $ sudo service mongod restart
   ~~~

3. 进入命令行

   ~~~shell
   mongo
   ~~~

4. 配置

   ~~~shell
   $ sudo vi /etc/mongod.conf
   ~~~

   > 注意这个配置文件的拓展名虽然是`conf`，但它的语法是`yaml`

   - 使MongoDB能够被远程访问

     `net.bindIp` 改为`0.0.0.0`

   - 使MongoDB以进程形式启动

     `processManagement.fork`改为`true`



## Nginx

1. 安装

   ~~~shell
   $ sudo yum install -y nginx
   ~~~

2. 启动

   ~~~shell
   $ sudo service nginx start
   ~~~

3. 开机启动

   ~~~shell
   $ sudo chkconfig nginx on
   ~~~

4. 重启

   ~~~shell
   $ sudo service nginx restart
   ~~~

5. 配置

   ~~~shell
   $ sudo vi /etc/nginx/nginx.conf
   ~~~


## NodeJS

1. 下载、安装

   ~~~shell
   $ sudo yum install -y nodejs
   ~~~

2.  npm安装n组件，升级到最新稳定版

   ~~~shell
   $ sudo npm install -g n
   $ sudo n stable
   ~~~

3. 安装完成

   ~~~shell
   $ node -v
   $ npm -v
   ~~~




## Maven

~~~shell
$ sudo yum install -y maven
~~~



## Git

1. 安装

   ~~~shell
   $ sudo yum install -y git
   ~~~

2. 安装完成

   ~~~shell
   $ git --version
   ~~~

3. 保存帐号密码

   ~~~shell
   $ git config --global credential.helper store
   ~~~

   ​

## Zookeeper

1. 下载

   ~~~shell
   $ cd /opt
   $ wget http://mirrors.hust.edu.cn/apache/zookeeper/zookeeper-3.4.13/zookeeper-3.4.13.tar.gz
   ~~~

2. 解压

   ~~~shell
   $ tar zxvf zookeeper-3.4.13.tar.gz
   ~~~



3. 生成配置文件

   ~~~shell
   $ cp /opt/zookeeper-3.4.13/conf/zoo_sample.cfg /opt/zookeeper-3.4.13/conf/zoo.cfg
   ~~~
   



4. 启动

   ~~~shell
   $ /opt/zookeeper-3.4.13/bin/zkServer.sh start
   ~~~

   - 停止

     ~~~shell
     $ /opt/zookeeper-3.4.13/bin/zkServer.sh stop
     ~~~

   - 重启

     ~~~shell
     $ /opt/zookeeper-3.4.13/bin/zkServer.sh restart
     ~~~



## RabbitMQ - Erlang

1. 下载、安装

   ~~~shell
   $ sudo vi /etc/yum.repos.d/rabbitmq-erlang.repo
   ~~~

   ~~~
   [rabbitmq-erlang]
   name=rabbitmq-erlang
   baseurl=https://dl.bintray.com/rabbitmq/rpm/erlang/21/el/7
   gpgcheck=1
   gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
   repo_gpgcheck=0
   enabled=1
   ~~~

   ~~~shell
   $ sudo yum install -y erlang
   ~~~

2. 验证安装成功

   ~~~shell
   $ erl
   ~~~

   ~~~
   > halt().
   ~~~



## RabbitMQ - Server

1. 下载、安装

   ~~~shell
   $ sudo rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
   $ sudo vi /etc/yum.repos.d/rabbitmq-server.repo
   ~~~

   ~~~
   [bintray-rabbitmq-server]
   name=bintray-rabbitmq-rpm
   baseurl=https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.7.x/el/7/
   gpgcheck=0
   repo_gpgcheck=0
   enabled=1
   ~~~

   ~~~shell
   $ sudo yum install -y rabbitmq-server
   ~~~


2. 验证安装成功 

   ~~~shell
   $ rabbitmq-server start
   ~~~

3. 开机启动

   ~~~shell
   $ chkconfig rabbitmq-server on
   ~~~

4. 启用Web管理后台

   ~~~shell
   $ rabbitmq-plugins enable rabbitmq_management
   ~~~

   > 至此，可以通过`127.0.0.1:15672`来管理RabbitMQ



5. 创建用户

   ~~~shell
   $ rabbitmqctl add_user yourUsername yourPassword 
   ~~~

   > 注意，这样创建的用户暂时是没有任何`Can access virtual hosts `的，可以去管理后台为其设置



6. 为用户授权

   授权分为授予`tags`和授予`permission`，一般来说授予`tags`往往够用

   ~~~shell
   $ rabbitmqctl set_user_tags yourUsername administrator
   ~~~

   其中，`administrator`是一种`tags`，RabbitMQ中，`tags`有5种

   management ：访问 management plugin；

   policymaker ：访问 management plugin 和管理自己 vhosts 的策略和参数；

   monitoring ：访问 management plugin 和查看所有配置和通道以及节点信息； 

   administrator ：一切权限；  

   None ：无配置  



## Elasticsearch

1. 下载、安装

   ~~~shell
   $ sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
   $ sudo vi /etc/yum.repos.d/elasticsearch.repo
   ~~~

   ~~~
   [elasticsearch-6.x]
   name=Elasticsearch repository for 6.x packages
   baseurl=https://artifacts.elastic.co/packages/6.x/yum
   gpgcheck=1
   gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
   enabled=1
   autorefresh=1
   type=rpm-md
   ~~~

   ~~~shell
   $ sudo yum install -y elasticsearch
   ~~~
