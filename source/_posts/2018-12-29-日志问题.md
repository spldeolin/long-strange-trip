---
title: 日志问题

date: 2018-12-29 17:05

updated: 2018-12-29 17:05

tags:
- Log4j2

categories: Java

permalink: log
---



## 简介

这篇POST将会介绍

- Java有哪些日志框架
- 日志门面和日志实现的区别、日志适配器是什么



## Java中的日志框架

Java世界中，有非常多的日志框架，它们大概可以分为3个阵营

![](/images/log-01.png)



## 日志框架的演变

1. Log4j

   Apache的Log4j早在2001就有了，作为一个第三方库，它在远古时代被广泛使用，性能比日后出现的其他第三方库要低。



2. JUL

   JUL指的是`java.util.logging`类，它在JDK1.4被加入，它功能性不如Log4j，性能也不太好。



3. Apache Commons Logging （官方简称为JCL）

   现在，有两个日志框架了——Log4j和JUL，他们都是日志实现。为了不让其他第三方库与这两个日志框架产生耦合，Apache推出了一个日志门面——JCL。

   JCL的出现，使得所有希望打印日志的地方，调用JCL提供的API就可以了，不用关注这条日志是由哪种实现打印的，实现了解耦。

   至此，Java世界只有一个流行的日志门面。



4. SLF4J、Logback等

   然而，JCL提供的API很容易让开发者写出性能不好的代码。于是，SLF4J诞生了，它也是日志门面，提供的API比JCL更合理一些。

   Logback是Log4j的继承，性能比后者更好，同时，还出现了SLF4J-nop、SLF4J-simple等其他日志实现。

   至此，Java世界存在了2个流行的日志门面和许多常用的日志实现了，也就是说，出现了两大阵营



5. 日志适配器

   两大阵营各自日志门面无法适配到另一个阵营的日志实现，第三方库的日志耦合问题又产生了。这次，SLF4J的作者做了许多日志适配器，让两个日志门面可以适配到所有常用的日志实现上。

   至此，开发者终于可以调着SLF4J的API，写着Log4j或是Logback的配置文件，日志问题告一段落。



6. Log4j2

   Apache终于推出了Log4j2，它跟Log4j完全不同，它模仿`SLF4J/Logback`的设计模似，自成一个阵营，Log4j-api对应SLF4J，Log4j-core对应Logback。Log4j2无论是门面还是实现，性能都非常强。Log4j2的出现还带来了大量的日志适配器，为了能跟已经存在的两个日志阵营适配起来。

   至此，三大阵营的关系是这样的

   ![img](/images/log-02.jpg)

图片来源 - [Java日志全解析（上） - 源流](https://zhuanlan.zhihu.com/p/24272450)



## 选用日志框架

日志框架那么多，可以按照以下的思路来选择它们

1. 选择一个性能好的日志实现

   推荐Logback或Log4j2-core，选择哪个可能取决你熟悉哪种实现的配置文件写法。

2. 选择若干个日志适配器

   确保上图的三个日志门面都能跟你选择的日志实现连接起来

3. 不需要选择日志门面加入你的项目

   除非你项目依赖的第三方库真的非常少，否则的话，第三方库们十有八九提供了3个日志门面

4. 选一个你喜欢的日志门面来调用它

   ~~~java
   public class Demo {
       // JCL
       private static final org.apache.commons.logging.Log jcl = org.apache.commons.logging.LogFactory.getLog(Demo.class);
       
   	// SLF4J
       private static final org.slf4j.Logger slf4j = org.slf4j.LoggerFactory.getLogger(Demo.class);
       
       // Log4j2-api
       private static final org.apache.logging.log4j.Logger log4j2 = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);
   }
   ~~~

   如果项目引入了`Lombok`，那事情就更简单了

   ~~~java
   // @CommonsLog
   // @SLF4J
   @Log4j2
   public class Demo {
       
   }
   ~~~
