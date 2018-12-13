---
title: Log4j2的yaml配置

date: 2018-04-05 14:28:00

updated: 2018-12-12 21:30:00

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
        level: all
        onMatch: ACCEPT
        onMismatch: DENY
      PatternLayout:
        charset: utf-8
        # 追加“%t”可打印线程名
        pattern: "%d{HH:mm:ss} %p [%c{1}:%L]%X{insignia} - %m%n"

  # 声明了两个logger，都是异步且显示行号的，都只打印到console
  # root logger：只打印error及以上级别的日志
  # com.spldeolin logger：打印所有日志，已被root logger打印过的日志不再答应
  Loggers:
    AsyncRoot:
      level: error # 修改为“debug”可查看所有类的DEBUG级日志
      includeLocation: true
      AppenderRef:
      - ref: console
    AsyncLogger:
      name: com.spldeolin
      level: all
      additivity: false
      includeLocation: true
      AppenderRef:
      - ref: console
```



这套配置只会在IDE打印console日志，不会生成log文件，因为开发环境往往只在IDE上运行项目的；



为了精简长度，不显示年月日信息，线程名不再打印，日志级别缩写为首字母。



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

~~~yaml
Configuration:
  status: warn

  Properties:
    Property:
    - name: log.dir
      value: # TODO 生产环境 日志目录名
    - name: log.name
      value: # TODO 生产环境 日志文件名
    - name: pattern
      value: "%d{yyyy-MM-dd HH:mm:ss} %p [%t][%c{1.}:%L]%X{insignia} - %m%n"

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

  # 声明了一个logger，异步且显示行号，打印所有级别
  # 打印到console，infoFile，debugFile这三个Appenders，Appenders各自的level再过滤一次打印级别
  Loggers:
    AsyncRoot:
      level: all
      includeLocation: true
      AppenderRef:
      - ref: console
      - ref: infoFile
      - ref: debugFile
~~~





## Spring Boot需要特别注意的地方

对于SpringBoot1.5.x，需要追加一个依赖

~~~xml
<dependency>
	<groupId>com.fasterxml.jackson.dataformat</groupId>
	<artifactId>jackson-dataformat-yaml</artifactId>
    <version>2.9.4</version>
</dependency>
~~~



而对于SpringBoot2.x.x版本，可以追加另外一个依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
</dependency>
~~~



一旦漏了相关依赖，Spring Boot无法识别log4j2.yml文件了