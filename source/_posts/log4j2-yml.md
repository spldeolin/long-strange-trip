---
title: Log4j2的yaml配置
date: 2018-04-05 14:28:00
tags: Log4j2
categories: Java
permalink: log4j2-yml
---

## 适合开发环境的配置

```yaml
########################
##                    ##
##     Devlopment     ##
##                    ##
########################

Configuration:
  status: warn

  Appenders:
    Console:
      name: console
      target: SYSTEM_OUT
      ThresholdFilter:
        # 修改为“debug”可查看DEBUG级日志
        level: info
        onMatch: ACCEPT
        onMismatch: DENY
      PatternLayout:
        charset: utf-8
        # 追加“%t”可打印线程名
        pattern: "%highlight{%d{HH:mm:ss} %p{DEBUG=D,INFO=I,WARN=W,ERROR=E}} [%c{1}:%L]%X{insignia} - %m%n"

  Loggers:
    Root:
      level: all
      AppenderRef:
      - ref: console
```



这套配置只会在IDE打印console日志，不会生成log文件，因为开发环境往往只在IDE上运行项目的；



为了精简长度，年月日、调试时基本不会长时间跨越好几天启动着项目，线程名不再打印，日志级别缩写为首字母。



## 适合测试环境的配置

~~~yaml
Configuration:
  status: warn

  Properties:
    Property:
    - name: log.dir
      value: # TODO 测试日志目录名
    - name: log.name
      value: # TODO 测试日志文件名
    - name: pattern
      value: "%highlight{%d{yyyy-MM-dd HH:mm:ss} %p{DEBUG=D,INFO=I,WARN=W,ERROR=E}} [%t][%c{1.}:%L]%X{insignia} - %m%n"

  Appenders:
    # 控制台
    Console:
      name: console
      target: SYSTEM_OUT
      ThresholdFilter:
        level: info
        onMatch: ACCEPT
        onMismatch: DENY
      PatternLayout:
        charset: utf-8
        pattern: ${pattern}

    RollingFile:
    # INFO及以上日志
    - name: infoFile
      ignoreExceptions: false
      fileName: ${sys:log.dir}/${sys:log.name}-INFO+.log
      filePattern: "${sys:log.dir}/${date:yyyy-MM}/${sys:log.name}-%d{yyyy-MM-dd}-INFO+.log.gz"
      append: true
      ThresholdFilter:
        level: info
        onMatch: ACCEPT
        onMismatch: DENY
      PatternLayout:
        charset: utf-8
        pattern: ${pattern}
      Policies:
        TimeBasedTriggeringPolicy:
          modulate: true
          interval: 1
        SizeBasedTriggeringPolicy:
          size: "128 MB"
      DefaultRolloverStrategy:
        max: 1000
    # DEBUG及以上日志
    - name: debugFile
      ignoreExceptions: false
      fileName: ${sys:log.dir}/${sys:log.name}-DEBUG+.log
      filePattern: "${sys:log.dir}/${date:yyyy-MM}/${sys:log.name}-%d{yyyy-MM-dd}-DEBUG+.log.gz"
      append: true
      ThresholdFilter:
        level: debug
        onMatch: ACCEPT
        onMismatch: DENY
      PatternLayout:
        charset: utf-8
        pattern: ${pattern}
      Policies:
        TimeBasedTriggeringPolicy:
          modulate: true
          interval: 1
        SizeBasedTriggeringPolicy:
          size: "128 MB"
      DefaultRolloverStrategy:
        max: 1000

  Loggers:
    Root:
      level: all
      AppenderRef:
      - ref: console
      - ref: infoFile
      - ref: debugFile
~~~



可以看到，测试环境的日志输出`PatternLayout`会比开发环境详细很多，主要是为了能最大程度保留**每个BUG的案发现场**。



另外，这套配置会打印console日志，也会生成log文件；

项目将会生成5个日志文件，分别记录了——

- INFO级及更严重等级的日志
- DEBUG级及更严重等级的日志
- 每次请求的执行时间；

日志文件文件超过128MB或者已经过时了1天以上，会被压缩到各自的gz压缩文件之中，减少磁盘压力。

投入使用一段时间后，日志目录结构将会是如下的情况

opt

- app-logs
  - app-INFO+.log
  - app-DEBUG+.log
  - 2018-04
    - app-2018-04-03-INFO+.log.gz
    - app-2018-04-03-DEBUG+.log.gz
    - app-2018-04-04-INFO+.log.gz
    - app-2018-04-04-DEBUG+.log.gz

比较容易管理和归档。



## 设计生产环境的配置

Deolin目前还在总结中... :)



## Spring Boot需要特别注意的地方

需要追加一个依赖

~~~xml
<dependency>
	<groupId>com.fasterxml.jackson.dataformat</groupId>
	<artifactId>jackson-dataformat-yaml</artifactId>
    <version>2.9.4</version>
</dependency>
~~~

一旦漏了这个依赖，Spring Boot就不认识log4j2.yml了