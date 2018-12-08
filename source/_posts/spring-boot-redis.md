---
title: Spring Boot集成Redis

date: 2018-04-19 14:42:00

updated: 2018-04-19 14:42:00

tags:
- Spring Boot
- Redis

categories: Java

permalink: spring-boot-redis
---

## 简介

Redis是一种NoSQL，与MongoDB不同是，数据是保存在内存中的。



## Window下安装Redis

### 下载

https://github.com/MicrosoftArchive/redis/releases

### 启动Redis

~~~shell
$ cd C:\java-development\redis-x64-3.0.504
$ redis-server redis.windows.conf
~~~

> 可以将启动命令作成BAT文件，方便以后启动

至此，Redis服务启动完毕。

### 图形化管理工具

https://redisdesktop.com

### 设置密码

编辑解压目录下的`redis.windows.conf`，追加——

~~~
requirepass root
~~~



## 集成Redis

### pom追加依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
~~~

> 这是个官方starter，所以无需指定版本坐标

### 配置application.yml

~~~yaml
spring:

  redis:
    host: localhost
    port: 6379
    password: root
    pool:
      max-active: 8
      max-wait: -1
      max-idle: 8
      min-idle: 0
~~~

至此，可以在Spring Boot项目中使用Redis了。



## 使用Redis

Spring提供的`org.springframework.data.redis.core.RedisTemplate`不算特别好用，所以封装它一下。

参考Deolin以前的POST——[序列化工具](http://spldeolin.com/posts/spring-redis/#SerializationUtil)、[存取Redis缓存的API](http://spldeolin.com/posts/spring-redis/#RedisCache)