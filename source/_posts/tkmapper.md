---
title: Mybatis 通用Mapper增强
date: 2018-01-03 08:52:00
tags:
- Spring
- Mybatis
categories: Java
permalink: tkmapper
---

## 事前准备

确保项目集成了Spring与Mybatis

## 追加依赖

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>3.4.6</version>
</dependency>
```

## 追加自定义通用Mapper

```java
/**
 * BaseMapper接口：使mapper包含完整的CRUD方法
 * ConditionMapper接口：使mapper支持Condition类型参数
 * MySqlMapper接口：使mapper支持MySQL特有的批量插入和返回自增字段
 * IdsMapper接口：使mapper支持批量ID操作
 *
 * @param <T> 实体类.class
 */
public interface CommonMapper<T> extends BaseMapper<T>, ConditionMapper<T>, MySqlMapper<T>, IdsMapper<T> {}
```

这里可以根据项目需求，自己定制。所有接口可以参照[Mapper接口大全](https://mapperhelper.github.io/all/ "Mapper接口大全")

## 修改Mybatis的MapperScannerConfigurer

改变spring-mybatis.xml中的org.mybatis.spring.mapper.MapperScannerConfigurer

```xml
<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--mapper接口所在位置-->
    <property name="basePackage" value="io.spldeolin.bestpractice.mapper" />
    <!--这里有个关注点：自定义Mapper不能放在mapper包里面-->
    <property name="properties" value="mappers=com.spldeolin.bestpractice.bean.CommonMapper" />
</bean>
```

## 使用

至此，每一个mapper接口（如UserMapper，OrderMapper等），都可以继承`通用Mapper`，从而可以少写很多很多的单表操作SQL文。如下所示

~~~java
public interface UserMapper extends CommonMapper<User> {}
~~~

~~~java
@Autowired
private UserMapper userMapper;

public List<User> listALLUsers() {
    return userMapper.selectAll();
}

~~~

