---
title: 分布式日志：使用Mongo收集Log4j2日志

date: 2019-01-04 10:40:00

updated: 2019-01-04 11:13:00

tags:
- Log4j2
- MongoDB

categories: Java

permalink: log4j2-mongo-appender
---



## 简介

在过去，服务端的应用往往会把日志打印到`.log`文件中。

如果日志量小或是没有分析日志的需求，这样的做法是完全可以的，但是，一旦需要分析日志，解析`.log`会十分不方便。

另外，在集群情况下，每个节点都将日志文件生成在自己所在的服务器，日志分散在各处，不好管理。

这篇POST将会改造Log4j2的配置，集成Mongo用于收集日志



## 追加相关依赖

示例代码基于的SpringBoot版本为`2.1.1.RELEASE`，如果是1.5.X版本，相关依赖会有所不同

~~~xml
<!-- 引入springboot mongo -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>

<!-- 引入disruptor，这个依赖能使Log4j2异步打印日志 -->
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.4.2</version>
</dependency>

<!-- 引入log4j2的nosql模块 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-nosql</artifactId>
    <version>2.9.1</version>
</dependency>

<!--
	降低log4j2的core模板版本
    SpringBoot为2.1.1RELEASE时，
    spring-boot-starter-log4j2依赖提供的log4j-core版本为2.11.1，
    需要降至与nosql模块一致，才能不报错。
    1.5.X版本没有这个问题
-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.9.1</version>
</dependency>
~~~



## Log4j2 声明nosql appender

~~~yaml
Appenders:

  # mongo appender
  NoSQL:
    name: mongo112233
    bufferSize: 10
    MongoDb:
      server: 192.168.2.2
      port: 27017
      username: guest
      password: guest
      databaseName: your_app_name
      collectionName: log
~~~



## Log4j2 logger使用nosql appender

~~~yml
Loggers:

  AsyncRoot:
    level: info
    includeLocation: true
    AppenderRef:
      - ref: console
      - ref: mongo112233
~~~



至此，配置完毕，项目能使用Mongo收集Log4j2的日志了。

使用[Navicat Premium 12](https://www.navicat.com/en/download/navicat-premium)连接Mongo，确认一下效果。

![](/images/log4j2-mongo-appender-01.png)



Mongo正确地收集了需要的日志，Navicat直观地展示了日志数据。

值得注意的是，Log MDC的信息，存放在`ContextMap`字段中。



## 集群问题

上诉做法可以让集群中每个节点的日志都收集在统一的地方，但是到目前为止还无法区分某条日志是有那个节点产生的，所以需要引入一个**运行时变量**。

首先，在log4j2.yml中追加配置

~~~yaml

~~~



现在，我们声明了一个`server`变量名，接着，在运行命令中追加这个变量

~~~shell
$ java -jar your-app.jar -Dserver=poiont1
~~~



然后，在mongo appender中引用server变量

~~~yaml
Appenders:

  # mongo appender
  NoSQL:
    name: mongo
    bufferSize: 10
    MongoDb:
      server: 192.168.2.2
      port: 27017
      username: guest
      password: guest
      databaseName: your_app_name
      collectionName: log_{server}
~~~



至此，应用每次启动都可以指定`server`，这个`server`会动态地拼接成mongo的collection，这样一来，应用启动时便可决定这个节点的日志会收集到哪个collection中，实现了集群日志的区分。



## 性能问题

为了方便定位BUG，每条日志都会记录行号，行号被收集在Mongo的 `source.lineNumber`字段中。

事实上，获取日志的位置信息是一种开销非常高的行为，[Apache官方](http://logging.apache.org/log4j/log4j-2.2/manual/appenders.html#AsyncAppender)也指出*“Extracting location is an expensive operation (it can make logging 5 - 20 times slower).”*

生产环境下，往往需要关闭这个行为

~~~yaml
Loggers:

  AsyncRoot:
    level: info
    includeLocation: false # 不再收集行号
    AppenderRef:
      - ref: console
      - ref: mongo112233
~~~



将`includeLocation`设置为`false`后，数据的整个`source`都为null了。



## 使用其他的Appender

当项目大到会产生海量的日志时，可以考虑用Hbase收集日志；或者对日志有强烈的分析需求时，可以考虑用Logstash收集日志，结合ES和Kibana进行管理。



## 源码

[spldeolin](https://github.com/spldeolin) / [log4j2mongo](https://github.com/spldeolin/log4j2mongo)