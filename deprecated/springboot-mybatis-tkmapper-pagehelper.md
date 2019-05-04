---
title: Spring Boot集成Mybatis、通用Mapper和PageHelper

date: 2018-04-01 16:45

updated: 2018-04-01 16:45

tags:
- Spring Boot
- Mybatis

categories: Java

permalink: springboot-mybatis-tkmapper-pagehelper
---

## pom追加所有的依赖

```xml
<!--MySQL-->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.45</version>
</dependency>
<!--Druid-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.9</version>
</dependency>
<!--Mybatis-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.1</version>
</dependency>
<!--通用Mapper-->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.2.4</version>
</dependency>
<!--PageHelper-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.3</version>
</dependency>
```

## 配置application.yml

~~~yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/app?characterEncoding=UTF-8&useUnicode=true&useSSL=false&allowMultiQueries=true
    username: root
    password: root
    druid:
      initialSize: 2
      minIdle: 2
      maxActive: 30

mapper:
  mappers:
   - tk.mybatis.mapper.common.Mapper
  not-empty: false
  identity: MYSQL

pageHelper:
  helperDialect: mysql
  reasonable: true
  supportMethodsArguments: true
  params: count=countSql
~~~

> mapper系的配置比较容易遗漏，一旦遗漏，启动时是不会报错，而是运行期间调用DAO时，报一些不太好定位到问题所在的错误。
>
> org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.builder.BuilderException: Error invoking SqlProvider method (tk.mybatis.mapper.provider.ConditionProvider.dynamicSQL).  

至此，配置完成，直接投入使用即可。

## 演示

测试类

~~~java
@Autowried
private OneCookieMapper oneCookieMapper;

@Test
public void test() {
    log.info(oneCookieMapper.selectAll().toString());
}
~~~

DAO

~~~java
@org.apache.ibatis.annotations.Mapper
public interface OneCookieMapper extends Mapper<OneCookie> {}
~~~

其中，通用Mapper建议搭配Mybatis Generator使用。

