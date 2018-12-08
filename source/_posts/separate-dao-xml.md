---
title: Mybatis DAO与xml文件分离

date: 2018-01-04 19:51:00

updated: 2018-01-04 19:51:00

tags:
- Spring
- Mybatis

categories: Java

permalink: separate-dao-xml
---

## 简介

对于一个Maven构建的项目，不建议将非Java文件放在src/main/java目录下。

原因首先是不规范，其次是对于一部分IDE，默认是**只**编译该目录下的java文件的，需要专门在pom.xml中配置&lt;resources&gt;标签，否则报这样的错

> org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)

所以需要将dao接口与对应xml文件分离到不同的目录下。

## 分离步骤

首先，dao接口肯定是不能配置在mybatis-config.xml的&lt;resources&gt;标签中了，原因如下——

~~~xml
 <!--
	将sql映射注册到全局配置中

	mapper 单个注册（mapper如果多的话，不太可能用这种方式）
            resource：引用类路径下的文件
            url：引用磁盘路径下的资源
            class，引用接口

	package 批量注册（基本上使用这种方式）
            name：mapper接口与mapper.xml所在的包名
-->
    <mappers>

        <!-- 第一种：注册sql映射文件-->
        <mapper resource="com/spldeolin/mapper/UserMapper.xml" />

        <!-- 第二种：注册接口   sql映射文件必须与接口同名，并且放在同一目录下-->
        <!--<mapper class="com.spldeolin.mapper.UserMapper" />-->

        <!-- 第三种：注册基于注解的接口  基于注解   没有sql映射文件，所有的sql都是利用注解写在接口上-->
        <!--<mapper class="com.spldeolin.mapper.TeacherMapper" />-->

        <!-- 第四种：批量注册  需要将sql配置文件和接口放到同一目录下-->
        <!--<package name="com.spldeolin.mapper" />-->

    </mappers>
~~~

总之就是不行。



其次，需要在sqlSessionFactory.mapperLocations指定mapper.xml的路径，在mapperScannerConfigurer.basePackage指定dao的包名。

*spring-mybatis.xml的片段*

~~~xml
	<bean id="sqlSessionFactory"
            class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource" />
        <!--Mybatis配置文件-->
        <property name="configLocation"
                value="classpath:mybatis-config.xml" />
        <!--mapper.xml所在位置-->
        <property name="mapperLocations" value="classpath:mapper/*Mapper.xml" />
        <!--指定需要使用别名的PO类所在的包-->
        <property name="typeAliasesPackage"
                value="com.spldeolin.demoapp.po" />
    </bean>

    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--mapper接口所在的包-->
        <property name="basePackage" value="com.spldeolin.demoapp.dao" />
    </bean>
~~~

*mybatis-config.xml最终的样子*

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 其他全局配置 -->
    <settings>
        <setting name="logImpl" value="LOG4J2" />
        <setting name="cacheEnabled" value="true" />
    </settings>

    <!--分页插件-->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <property name="dialect" value="mysql" />
            <property name="offsetAsPageNum" value="true" />
            <property name="rowBoundsWithCount" value="true" />
            <property name="pageSizeZero" value="true" />
            <property name="reasonable" value="false" />
            <property name="returnPageInfo" value="check" />
            <property name="params" value="pageNum=start;pageSize=limit;" />
        </plugin>
    </plugins>

</configuration>
~~~

至此，分离完成。完成后的项目目录结构将会是这样的——![](/images/1139226-20180105095728471-1807856072.png)
