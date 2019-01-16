---
title: Spring Boot集成MongoDB

date: 2018-04-05 08:10

updated: 2018-04-05 08:10

tags: 
- Spring Boot
- MongoDB

categories: Java

permalink: spring-boot-mongodb
---



## 简介

MongoDB是一种NoSQL，保存的数据是以文件的形式存储在磁盘中的。

MongoDB可以认为是一个保存Key-Value的数据库，如果能总是知道Key，或者说Value的数据类型多变的场景，建议使用MongoDB。

## Windows下安装MongoDB

### 官网

​	https://www.mongodb.com/download-center?jmp=nav#community

### 建立存储数据的目录

> 比如：`C:\\mongodb-dbpath`

### 安装完毕后启动MongoDB

~~~shell
$ cd C:\Program Files\MongoDB\Server\3.6\bin
$ mongod --dbpath C:\mongodb-dbpath
~~~

> 可以将启动命令作成BAT文件，方便以后启动

至此，MongoDB服务启动完毕。

### 启动管理台

~~~shell
$ cd C:\Program Files\MongoDB\Server\3.6\bin
$ mongo
~~~

> 需要MongoDB服务启动的情况下，管理台才能启动

### 建立指定数据库

> 这里的“数据库”，对应Mysql的说法就是scheme

~~~shell
> use try_mongodb;
~~~

> 需要有数据，才能在`> show dbs;`中看到新建的数据库

### 创建用户

~~~shell
> db.createUser({user:"root",pwd:"root",roles:[{role:"readWrite",db:"try_mongodb"}]});
~~~

> 网上提到的`db.addUser()`命令在当前版本已经不可用了

### 验证用户是否被成功创建

~~~shell
> use try_mongodb;
> db.auth('root', 'root');
~~~

至此，MongoDB在Windows下安装完毕，下面去Spring Boot那里配置。

## 集成MongoDB

### pom追加依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
~~~

> 这是个官方starter，所以无需指定版本坐标

### 配置application.yml

~~~yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:root@localhost:27017/try_mongodb
~~~

至此，可以在Spring Boot项目中使用MongoDB了。

## 使用MongoDB

Deolin感觉Spring Data提供的MongoTemplate比RedisTemplate好用很多，所以也就无需一层API来代理MongoTemplate了，直接调用即可。

示例

~~~java
@Autowired
private MongoTemplate mongoTemplate;

@Test
public void save() {
    mongoTemplate.save(User.builder().id(233).name("Deolin").age(18).build());
}

@Test
public void get() {
    Query quest = new Query();
    query.addCriteria(Criteria.where("id").is(233));
    log.info(mongoTemplate.find(query, User.class));
}

~~~

文档类

~~~java
@Data
@Document("user")
public class User {

    @org.springframework.data.annotation.Id
    private Long id;

    private String name;

    private Integer age;

}
~~~

