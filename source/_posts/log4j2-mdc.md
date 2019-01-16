---
title: Log4j2基于MDC显示当前登录者的信息

date: 2018-05-06 11:24

updated: 2018-05-06 11:24

tags:
- Log4j2

categories: Java

permalink: log4j2-mdc
---



## 简介

对于一个Web服务来说，如果能够在每个业务日志前面，都显示系统当前登录者的信息，那么对于数据分析或是BUG复现来说，是在方便不过了。

在业务代码中每次通过API获取登录者信息，拼接成一条日志固然能实现上面提到的需求，但是，这种做法或产生大量的重复代码。

这篇POST介绍了如何借助MDC，优雅地在每行日志中显示当前登录者的信息。



## X占位符

在[官方文档](https://logging.apache.org/log4j/2.x/manual/layouts.html)的Pattern Layout章节的Patterns子章节下，可以看到对于`%X`占位符的描述。

描述中提到，存入`ThreadContext`的映射关系，能够输出到对应的`X`占位符中，这正好是显示登录者信息的理想实现方式。



## 实现

1. 修改log4j2配置文件的`PatternLayout.pattern`格式，追加`X`占位符

   ~~~
   "%highlight{%d{HH:mm:ss} %p{DEBUG=D,INFO=I,WARN=W,ERROR=E}} [%c{1}:%L] - %m%n"
   ~~~

   ~~~
   "%highlight{%d{HH:mm:ss} %p{DEBUG=D,INFO=I,WARN=W,ERROR=E}} [%c{1}:%L]%X{username} - %m%n"
   ~~~

   ​

2. 追加Java代码，在登录成功后，往`ThreadContext`存入映射关系

   ~~~java
   Subject subject = SecurityUtils.getSubject();
   try {
       subject.login(input.toToken());
   } catch (AuthenticationException e) {
       throw new ServiceException(e.getMessage());
   }
   ThreadContext.put("username", "[" + subject.getPrincipal() + "]");
   ~~~
   ​

3. 追加Java代码，在退出登录后，清除`ThreadContext`的映射关系

   ~~~java
   SecurityUtils.getSubject().logout();
   ThreadContext.remove("username");
   ~~~



至此，实现完毕。



## 效果

首先是匿名可访问接口的日志

![](/images/log4j2-mdc-01.png)

和原先没加`X`的时候一样，因为这时候`ThreadContext` 中不存在`username`



接着是登录接口的日志

![](/images/log4j2-mdc-02.png)

一旦登录成功，`username`就在日志中显示出来了。另外，`ControllerAspect`是个环绕切面，`开始处理...`前的日志不显示`username`，这也符合语义，毕竟只有处理完毕，用户才算登录了。



然后是认证才能访问接口的日志

![](/images/log4j2-mdc-03.png)

非常清晰，无论是切面中的日志，还是业务代码中的日志，都能显示`username`



最后是退出登录的日志

![](/images/log4j2-mdc-04.png)

跟登录接口同理，`...处理完毕`之后才不显示`username`