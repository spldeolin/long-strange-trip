---
title: Log4j2将特定的日志输出到专门的文件

date: 2018-04-26 17:25

updated: 2018-04-26 17:25

tags:
- Log4j2

categories: Java

permalink: specific-log-output-to-specific-file
---

## 简介

这篇POST一定程度上承接了*[利用Spring监听器打印每次请求的执行时间](http://spldeolin.com/posts/processing-time-log-listener/)*。

很多时候，我们需要把项目中一些特定的日志提取出来，生成一个专门的log文件。

Deolin将会介绍如何利用Log4j2做到这件事。



## 配置文件 

这里以yaml格式的配置为例，想达到的效果是 由**一个特定类**生成日志全部生成到专门的文件中。

1. 首先在`RollingFile`下追加一个新节点——

   ~~~yaml
       # 请求执行时间日志
       - name: processingTimeFile
         ignoreExceptions: false
         fileName: ${sys:log.dir}/${sys:log.name}-TIME.log
         filePattern: "${sys:log.dir}/${date:yyyy-MM}/${sys:log.name}-%d{yyyy-MM-dd}-TIME.log.gz"
         append: true
         Filters:
           ThresholdFilter:
             level: info
             onMatch: ACCEPT
             onMismatch: DENY
         PatternLayout:
           charset: utf-8
           pattern: "%m\t\t%d{yyyy-MM-dd HH:mm:ss.SSS}%n"
         Policies:
           TimeBasedTriggeringPolicy:
             modulate: true
             interval: 1
           SizeBasedTriggeringPolicy:
             size: "128 MB"
         DefaultRolloverStrategy:
           max: 1000
   ~~~

   这个示例中，**“特定的日志”**全是由一个类产生，线程号也没有什么必要打印，并且这批日志会统一到一个文件中，所以大幅精简了`PatternLayout`，仅仅是个建议。

2. `Loggers`下追加——

   ~~~yaml
       Logger:
       - name: com.spldeolin.beginningmind.component.ServletRequestProcessingTimeLogListener
         additivity: true
         level: all
         AppenderRef:
         - ref: processingTimeFile
   ~~~


   关键属性是`name`，这里不光可以写类的Reference，还可以写包的，只要是个全限定名的前片段就可以了。

   **所有以`name`开头的类中打印的日志，都会打印到`processingTimeFile`代表的`RollingFile`中。**



## 最终效果

![](/images/specific-log-output-to-specific-file-1.png)



显然，完全看不清（指的是格式看不请），解决方法是复制到Excel。

![](/images/specific-log-output-to-specific-file-2.png)



可以说是相当清晰了。不光如此，还可以借助Excel本身，筛查，排序，从而更好分析系统性能。