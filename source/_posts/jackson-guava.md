---
title: Jackson集成Guava Collection

date: 2018-09-26 08:40

updated: 2018-09-26 08:40

tags:
- Jackson
- Guava

categories: Java

permalink: jackson-guava
---



## 简介

Google Guava提供了几个额外的Collcetion，它们很实用，优点也非常多，具体的用法可以参考[官方文档](https://github.com/google/guava/wiki/NewCollectionTypesExplained)。

默认情况下，无法像Java Collection自带的类型那样直接序列为JSON字符串。

这篇POST将会介绍如果让Jackson能够正确地序列化Guava Collection类型。



## POM依赖

~~~xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-guava</artifactId>
    <version>2.9.7</version>
</dependency>
~~~

注意，`jackson-datatype-guava`需要与`jackson-core`、`jackson-datablind`等基础依赖的版本保持一致。



方便的是，Jackson提供了BOM用于统一管理所有依赖的版本，如下所示

~~~xml
<dependencies>
    <!--jackson-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
    <!--jackson (jsr310, guava, yaml)-->
    <dependency>
        <groupId>com.fasterxml.jackson.datatype</groupId>
        <artifactId>jackson-datatype-jsr310</artifactId>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.datatype</groupId>
        <artifactId>jackson-datatype-guava</artifactId>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-yaml</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <!--jackson (bom)-->
        <dependency>
            <groupId>com.fasterxml.jackson</groupId>
            <artifactId>jackson-bom</artifactId>
            <version>2.9.7</version>
        </dependency>
    </dependencies>
</dependencyManagement>
~~~



## 配置ObjectMapper

~~~java
ObjectMapper defaultObjectMapper = new ObjectMapper();

// others configure ...

defaultObjectMapper.registerModule(new GuavaModule());
~~~



## 演示

~~~java
Multimap<String, String> multimap = ArrayListMultimap.create();
multimap.put("111", "数字1");
multimap.put("111", "数字2");
multimap.put("111", "数字12313");
multimap.put("111", "数字");
multimap.putAll("adf", Lists.newArrayList("字母", "char"));

log.info(defaultObjectMapper.writeValueAsString(multimap));	
~~~



![](/images/jackson-guava-1.png)



## 为Spring Boot添加Guava Collection支持

有时候我们想要让Guava Collection作为请求方法的@RequestBody或是@ResponseBody，本质上是为内置的Jackson集成Guava Collcetion



~~~java
@Bean
public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {
    Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.json();

    builder.modules(new GuavaModule());

    // others configure ...

    return builder;
}
~~~



~~~java
@RestController
@RequestMapping("test")
public class TestController {
    
    @GetMapping("/table")
    Table<Long, Double, String> ln100() {
        Table<Long, Double, String> table = HashBasedTable.create();
        table.put(1L, 1.2, "阿斯顿");
        table.put(1L, 1.88, "阿古斯");
        return table;
    }
    
}
~~~



![](/images/jackson-guava-2.png)