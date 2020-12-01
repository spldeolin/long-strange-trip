---

title: 安装RocketMQ及其控制台

date: 2019-06-27 19:15

updated: 2019-06-28 08:15

tags:
- RocketMQ

categories: Linux

permalink: install-rocketmq

---



## 安装RocketMQ

1. **确保环境变量`JAVA_HOME`已经配置**

2. 下载源码并打包

   ~~~shell
   unzip rocketmq-all-4.4.0-source-release.zip
   cd rocketmq-all-4.4.0/
   ~~~

3. Maven打包

   ~~~shell
   mvn -Prelease-all -DskipTests clean install -U
   cd distribution/target/apache-rocketmq
   ~~~

4. 启动Name Server

   ~~~shell
   nohup sh bin/mqnamesrv &
   ~~~

5. 启动Broker之前**确保内存大于8G**，如果不足8G，需要编辑`bin/runbroker.sh`将其中的-Xmx等配置项删掉，否则无法启动

6. 启动Broker

   ~~~shell
   nohup sh bin/mqbroker -n localhost:9876 &
   ~~~

   

## 安装RocketMQ控制台

1. 拉取源码

   ~~~shell
   git clone https://github.com/apache/rocketmq-externals.git
   ~~~

2. Maven打包

   ~~~shell
   mvn -DskipTests clean install
   ~~~

3. 启动控制台，这是个Spring Boot的Web项目

   ~~~shell
   nohup java -jar target/rocketmq-console-ng-1.0.1.jar &
   ~~~

4. 通过日志可以看到，端口号是`8080`