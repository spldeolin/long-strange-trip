---
title: 为Mybatis配置多个数据源

date: 2018-05-09 19:20

updated: 2018-05-09 19:20

tags:
- Mybatis

categories: Java

permalink: mybatis-multi-datasource
---

## 简介

对于大项目而言，多数据源的必要性和好处不必多说。

Deolin将在这篇POST介绍一下如何利用多个平行配置文件，为Mybatis配置多个数据源，最终，希望达到以下的效果——

- 原mapper包拆分成不同子包，每个子包代表一个数据源。
- 通过配置映射关系，使得每个Mapper能通过所在的包能够自声明自己属于什么数据源。
- 解耦，业务层、控制层等感知不到多数据源这回事，依然是照常调用Mapper接口。



##集成流程 

### Mybatis

首先需要确保Mybatis被正确集成，如果在Spring Boot体系下，可以[参照这里](http://spldeolin.com/posts/spring-boot-mybatis-tkmapper-pagehelper)。



### JDBC配置

`application.yml`

~~~yaml
spring:
  datasource:
  	bm1:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/beginning_mind
      username: root
      password: root
      druid:
        initialSize: 2
        minIdle: 2
        maxActive: 30
    bm2:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/beginning_mind_2
      username: root
      password: root
      druid:
        initialSize: 2
        minIdle: 2
        maxActive: 30
~~~



### 建立第一个数据源的配置类

创建`DataSource1Config`

~~~java
import javax.sql.DataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import tk.mybatis.spring.annotation.MapperScan;

/**
 * 第一个数据源
 */
@Configuration
@MapperScan(basePackages = "com.spldeolin.beginningmind.dao.bm1", sqlSessionTemplateRef = "bm1SessionTemplate")
public class DataSource1Config {

    @Bean(name = "bm1DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.bm1")
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "bm1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("bm1DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/bm1/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "bm1TransactionManager")
    @Primary
    public DataSourceTransactionManager testTransactionManager(@Qualifier("bm1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "bm1SessionTemplate")
    @Primary
    public SqlSessionTemplate test1SqlSessionTemplate(
            @Qualifier("bm1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
~~~



其中——

- `@Primary`需要声明，第一个数据源作为主数据源。
- **如果项目集成了tk.mapper，`@MapperScan`必须是`tk.mybatis.spring.annotation.MapperScan`**

- `@ConfigurationProperties`的`prefix`属性填的是第一个数据源JDBC配置的前缀。

- `@MapperScan`的`basePackages`属性填的是该数据源内的表对应Mapper接口所在的全包名。
- `getResources()`方法的参数填的是上述Mapper接口对应的Mapper.xml文件所在的路径。
- `@MapperScan`的`sqlSessionTemplateRef`属性要与类中声明的`SqlSessionTemplate`的`@Bean` `name`一致。



### 建立其他数据源的配置类

创建`DataSource2Config`

~~~java
@Configuration
@MapperScan(basePackages = "com.spldeolin.beginningmind.dao.bm2", sqlSessionTemplateRef = "bm2SessionTemplate")
public class DataSource2Config {

    @Bean(name = "bm2DataSource")
    @ConfigurationProperties(prefix = "spring.datasource.bm2")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "bm2SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("bm2DataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/bm2/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "bm2TransactionManager")
    public DataSourceTransactionManager testTransactionManager(@Qualifier("bm2DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "bm2SessionTemplate")
    public SqlSessionTemplate test1SqlSessionTemplate(
            @Qualifier("bm2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
~~~



其中——

- 除了`@Primary`不能声明，其他地方与第一个数据源同理。
- 第三个、第四个...数据源与第二个数据源配置基本一致。



## 其他注意点

配置多数据源可能并不复杂，但对于一个既存项目来说，

一些持久层的配置可能在配置多数据源的时候出现各种各样的问题，这些问题在Spring Boot的AutoConfigure大环境下可能会非常难以定位和解决，所以，**建议在项目搭建之初，给多数据源预留配置空间。也就是说在一开始，最好把把第一个数据源的配置给写好**

其次，另一个比较容易出错的地方，应该是批量移动Mapper接口和Mapper.xml文件的时候，会出现一些引用问题。建议用IntelliJ IDEA的重构功能来完成这项移动工作。