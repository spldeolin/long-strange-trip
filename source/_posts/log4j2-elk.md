---
title: ELK收集Log4j2日志

date: 2019-02-22 10:17

updated: 2019-02-22 10:17

tags:
- Log4j2
- Elasticsearch

categories: Java

permalink: log4j2-elk
---

## 简介

这篇POST将会介绍

- 什么是ELK

- 在开发环境搭建ELK
- 使用ELK收集Log4j2日志（并提供一个简单的可运行的示例）



## 什么是ELK

ELK代表3种技术，是一种的流行的日志收集解决方案

**E**lasticsearch + **L**ogstash + **K**ibana

简单来说，Logstash用于接受来自不同微服务的日志，并保存到Elasticsearch；Elasticsearch用于持久化日志；Kibana是Elasticsearch的可视化工具。

可见将3者粗暴地理解为JDBC+MySQL+Navicat



## 在开发环境搭建ELK

1. 安装

   ~~~bash
   $ brew install elasticsearch
   $ brew install logstash
   $ brew install kibana
   ~~~

2. 启动Elasticsearch和Kibana

   ~~~shell
   $ brew services start elasticsearch
   $ brew services start kibana
   ~~~

3. 确认正常启动

   浏览器访问`127.0.0.1:9200`和`127.0.0.1:5601`

4. Logstash配置

   ~~~shell
   $ vi ~/log4j2elk-log.conf
   ~~~

   ~~~shell
   input {
     gelf {
       host => "127.0.0.1"
       port => 4560
       use_tcp => true
     }
   }
   
   output {
     elasticsearch {
       hosts => ["http://localhost:9200"]
       index => "log4j2elk-log-%{+YYYY-MM-dd}"
     }
   }
   ~~~

5. 启动logstash

   ~~~shell
   $ logstash -f ~/log4j2elk-log.conf
   
   ...
   [2019-02-21T16:52:59,761][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
   ~~~

   > 注意不要忘记`-f`



## 与服务对接

1. pom

   ~~~xml
       <dependency>
           <groupId>biz.paluch.logging</groupId>
           <artifactId>logstash-gelf</artifactId>
           <version>1.13.0</version>
       </dependency>
   ~~~

2. 为log4j2追加新的appender

   ~~~yaml
   Gelf:
   	name: logstash
   	host: 127.0.0.1
   	port: 4560
   	includeFullMdc: true # 收集每条日志的Log MCD数据
   ~~~

   

## 源码

[spldeolin](https://github.com/spldeolin) / [log4j2elk](https://github.com/spldeolin/log4j2elk)