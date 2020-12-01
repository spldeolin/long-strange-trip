---
title: Mybatis的相关组件的路径配置会去扫描子目录

date: 2018-03-08 10:10

updated: 2018-03-08 10:10

tags:
- Mybatis

categories: Java

permalink: mybatis-scan-packages-recursively
---

开门见山，例如

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
        <!--Mybatis配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml" />
        <!--mapper.xml所在位置-->
        <property name="mapperLocations" value="classpath:mapper/*Mapper.xml" />
        <!--model所在位置-->
        <property name="typeAliasesPackage" value="com.spldeolin.demo.model" />
    </bean>
```

在这里，
如果将某个mapper.xml移动到类似`mapper/aaa`的目录下，
或是将某个model移动到类似`com.spldeolin.demo.model.aaa`的包下，
或是将某个mapper接口移动到类似`com.spldeolin.demo.dao.aaa`的包下，
（当然，移动行为需要IDE的重构功能来确保Reference一致）
是**不会报错**的，因为框架会去扫描`mapperLocations`和`typeAliasesPackage`指定的目录的子目录。

------------

同样的，即便是在Mybatis上配置了通用mapper增强，情况也是如此。

```xml
    <bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--mapper接口所在位置-->
        <property name="basePackage" value="com.spldeolin.demo.dao" />
        <property name="properties" value="mappers=com.spldeolin.demo.util.bean.Mapper" />
    </bean>
```