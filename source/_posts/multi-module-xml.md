---
title: Maven多模块时Invalid bound statement报错的问题

date: 2019-03-31 10:34

updated: 2019-03-31 10:34

tags:
- Mybatis
- Maven

categories: Java

permalink: multi-module-xml
---

## 简介

写Mybatis Plus分页那篇POST的时候，发现了一个BUG，刚解决了，这里记录一下



## 源流

Deolin将项目分为了多个Maven Module，

在Spring Boot 启动方法所在Module的`dependencies`里面，依赖了所有其他模块。

其他模块中，有不少Mybatis的mapper.xml文件，

每次调用其他模块的Mapper方法时，都会抛出异常

~~~
Invalid bound statement
~~~



## 发生原因

除开 标签id和方法名不一致这样的粗心问题，`Invalid bound statement`引发的原因基本上是xml在启动时没有加载到内存中。

Deolin查询了一下资料发现，以下的写法，Spring Boot启动时只会加载本模块的xml文件

~~~yaml
  mapper-locations: classpath:mapper/*.xml
~~~



## 解决方法

需要在classpath后面追加一个`*`，Spring Boot才会去加载它们模块的xml文件

~~~yml
  mapper-locations: classpath*:mapper/*.xml
~~~





