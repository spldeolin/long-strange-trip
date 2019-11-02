---
title: Spring Boot的零散技巧

date: 2018-04-30 20:47

updated: 2018-04-30 20:47

tags:
- Spring Boot
- Spring

categories: Java

permalink: springboot-tips
---



## 简介

主要介绍平时使用Spring Boot过程的中小技巧，这些技巧都经过了Deolin的验证。



## 一览

### 保持项目启动

一个非Web的Spring Boot项目，启动成功后会直接退出。需要让项目保持启动的话，可以修改启动类

~~~java
@SpringBootApplication
public class DemoApplication implements CommandLineRunner {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        Thread.currentThread().join();
    }

}
~~~

如此一来，这个项目不需要集成spring-boot-starter-web，就可以运行定时任务了​。



### 额外监听路径​

Spring Boot内置了Tomcat，想要配置额外监听路径的话，可以追加一个配置组件

~~~java
@Component
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**").addResourceLocations("file:C:/demo-static/");
    }

}
~~~

用到了装饰器模式，这段代码的作用与下面这段Tomcat配置类似。

~~~xml
<Context path="/static" docBase="C:\\demo-static" debug="0" reloadable="true" />
~~~

另外，即便是将内置Tomcat换成Undertow，`WebConfig`依然能够生效。



### 简化启动类注解

启动类的类注解，只有`@SpringBootApplication`是必须的，其他的注解（如各种@Enable...注解）完全可以写在配置类上，方便管理。



### 指定404页面

Http404错误无法通过统一异常处理来捕获，需要有个控制器实现专门的接口

~~~java
@RestController
public class RedirectController implements ErrorController {

    public static final String NOT_FOUND_MAPPING = "/error";

    @RequestMapping(NOT_FOUND_MAPPING)
    public RequestResult notFound() {
        return RequestResult.failure(ResultCode.NOT_FOUND);
    }

    @Override
    public String getErrorPath() {
        return NOT_FOUND_MAPPING;
    }

}
~~~

这里的映射值必须是`/error`，否则报错。