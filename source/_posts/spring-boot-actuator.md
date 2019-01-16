---
title: Spring Boot集成actuator

date: 2018-05-02 13:32

updated: 2018-05-02 13:32

tags:
- Spring Boot

categories: Java

permalink: spring-boot-actuator
---

## 简介

actuator是一个用于打印项目运行时的各项指标的Spring Boot官方starter。

Spring Boot项目开启了这个功能后，我们可以通过调用actuator提供的HTTP请求来使用它。



## 追加依赖

~~~xml
<!--springboot actuator-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
~~~



## 追加配置

~~~yaml
management:
  security:
    enabled: false
  port: ${server.port}
  context-path: /actuator
  endpoints:
    shutdown:
      enabled: true
~~~



## 使用

actuator提供了许多可供访问的端点。以`/beans`端点为例，可以通过以下URL来访问到它

​	http://ip:`server.port`/`server.context-path`/`management.context-path`/beans



常用端点如下：

- `/bean` 查看ApplicationContext中的所有bean 
- `/configprops`  查看配置属性
- `/mappings` 查看所有控制器请求方法的URL映射
- `/metrics` 查看内存、线程、GC信息
- `/health` 查看MySQL、Redis、MongoDB等模块的健康指标
- `/trace` 最近100次HTTP请求的大致情报



## Shiro放行actuator端点请求

一般来说，非生产环境下，为了方便起见，actuator的端点请求最好无须认证就能访问，这可以在Shiro过滤器上追加一些配置

~~~java
    @Autowired
    private Environment environment;

    @Value("${management.context-path}")
    private String actuatorContextPath;

    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        // ...
        
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        
        // ...
        
        if (!ArrayUtils.contains(environment.getActiveProfiles(), "prod")) {
            // 非prod环境放行actuator端点请求
            if (StringUtils.isNotBlank(actuatorContextPath)) {
                filterChainDefinitionMap.put(actuatorContextPath + "/**", "anon");
            }
        }
        filterChainDefinitionMap.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        
        // ...
    }
~~~

