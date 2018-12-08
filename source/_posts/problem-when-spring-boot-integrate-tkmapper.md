title: Spring-boot整合通用Mapper增强时的问题
date: 2018-03-14 20:02:28

tags: Spring Boot

categories: Java

permalink: problem-when-spring-boot-integrate-tkmapper
---

## 问题重现

Spring Boot整合Mybatis时，需要让容器能够识别到Mapper接口。

有两种方式可供选择：

1. 在Mapper接口上声明`@org.apache.ibatis.annotations.Mapper`注解
2. 在Spring Boot的启动方法上声明`@MapperScan("com.xxx.dao")`

一般来说，Mapper接口过多的话，第二种方法比较合适。

当为项目引入tk.mapper依赖以后，上述做法会直接报错

> tk.mybatis.mapper.provider.base.BaseSelectProvider:Caused by: java.lang.InstantiationException: ...

## 应对方式

实际上，tk.mapper依赖中也提供了一个MapperScan注解类，也就是说，Spring Boot整合Mybatis + tk.mapper后，正确的MapperScan注解是`@tk.mybatis.spring.annotation.MapperScan("com.xxx.dao")`

## 第二种应对方式

两个MapperScan由于类名一模一样，所以非常容易搞错，一旦出错也不太容易定位到问题所在。

更合适的做法是可能是在**问题重现**中提到的第2.种方法，`@Mapper`只有Mybatis提供了，tk.mapper中没有，而且即使不整合tk.mapper，`@Mapper`也是需要的，这样一来就很难弄错了。

至于碰到了Mapper接口比较多的项目，加`@Mapper`这件事也完全可以由代码生成器代劳