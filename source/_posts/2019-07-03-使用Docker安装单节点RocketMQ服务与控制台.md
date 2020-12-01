---

title: 使用Docker安装单节点RocketMQ服务与控制台

date: 2019-07-03 10:23

updated: 2019-07-03 10:23

tags:
- MQ

categories: Java

permalink: docker-install-rocketmq

---

## namesrv和broker

1. 下载Docker镜像

   ~~~shell
   git clone https://github.com/apache/rocketmq-externals.git
   cd rocketmq-externals/
   ~~~

2. 安装镜像

   ~~~shell
   sh build-image.sh 4.5.1
   docker image ls
   ~~~

3. 创建namesrv container，并启动

   ~~~shell
   docker run -d -p 9876:9876 \
   -v /Users/deolin/rocketmq/data/namesrv/logs:/root/logs \
   -v /Users/deolin/rocketmq/data/namesrv/store:/root/store \
   --name rmqnamesrv -e "MAX_POSSIBLE_HEAP=100000000" \
   rocketmqinc/rocketmq:4.5.1 sh mqnamesrv
   
   docker container ls
   ~~~

4. 创建broker container，并启动

   ~~~shell
   docker run -d -p 10911:10911 -p 10909:10909 \
   -v /Users/deolin/rocketmq/data/broker/logs:/root/logs \
   -v /Users/deolin/rocketmq/data/broker/store:/root/store \
   --name rmqbroker --link rmqnamesrv:namesrv \
   -e "NAMESRV_ADDR=namesrv:9876" \
   -e "MAX_POSSIBLE_HEAP=200000000" \
   rocketmqinc/rocketmq:4.5.1 sh mqbroker
   
   docker container ls
   ~~~

5. 完毕



## RocketMQ控制台

1. 拉取镜像

   ~~~shell
   docker pull styletang/rocketmq-console-ng
   docker image ls
   ~~~

2. 创建container，并启动

   ~~~shell
   docker run \
   -e "JAVA_OPTS=-Drocketmq.namesrv.addr=172.17.0.3:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
   -p 8080:8080 -t styletang/rocketmq-console-ng
   ~~~



## 特别注意

1. 必须先启动`namesrv`，再启动`broker`，否则报错

2. 创建控制台的container时，`-Drocketmq.namesrv.addr`必须指定为`namesrv`所在container的IP

   可以通过以下命令来查看每个container的IP地址

   ~~~shellshe l
   docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
   ~~~

   

