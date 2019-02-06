---
title: 在SpringBoot中处理404错误

date: 2018-05-11 17:33

updated: 2018-12-10 08:48

tags:
- Spring

categories: Java

permalink: springboot-override-404
---

## 简介

对于一个Spring Boot Web项目而言，当浏览器访问一个不存在的接口时，会得到一个`Whitelabel Error Page`页面。这篇POST介绍了如何重写掉Spring Boot对请求404的处理方式。



## 实现方式

1. 修改`DispatcherServlet`的属性

   ~~~java
   @Configuration
   public class AnyConfigClass {
   
       /**
        * 请求404时不再转发到Whitelabel Error Page，而是抛出NoHandlerFoundException异常
        */
       @Bean
       DispatcherServlet dispatcherServlet() {
           DispatcherServlet dispatcherServlet = new DispatcherServlet();
           dispatcherServlet.setThrowExceptionIfNoHandlerFound(true);
           return dispatcherServlet;
       }
   
   }
   ~~~


2. 在Spring Boot容器能扫描到的类上追加`@EnableWebMvc`注解

   ~~~java
   @EnableWebMvc
   @Component
   public class AnyCompontent {
   
       // ...
   
   }
   ~~~



3. 至此，当外部访问请求404时，会抛出一个`NoHandlerFoundException`异常，使用统一异常处理捕获它即可令请求404时返回我们希望的返回值

   ~~~java
   @RestControllerAdvice
   public class ExceptionAdvance {
    
       /**
        * 404 找不到请求
        */
       @ExceptionHandler(NoHandlerFoundException.class)
       public RequestResult requestHandlingNoHandlerFound(NoHandlerFoundException ex) {
           return RequestResult
                   .failure(ResultCode.NOT_FOUND, "请求或资源不存在" + " [" + ex.getHttpMethod() + "]" + ex.getRequestURL());
       }
   
   }
   ~~~



## 其他

比以往的方式简单太多了...

源码可以参照[spldeolin](https://github.com/spldeolin) / [beginning-mind](https://github.com/spldeolin/beginning-mind)