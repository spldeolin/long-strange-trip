---
title: 阅读Spring Mvc Showcase源码后的收获

date: 2018-06-28 09:56:00

updated: 2018-06-28 09:56:00

tags:
- Spring

categories: Java

permalink: spring-mvc-showcase
---



## 简介

[Spring Mvc Showcase](https://github.com/spring-projects/spring-mvc-showcase)是Spring官方的示例项目，向开发者提供了关于Spring Web MVC的实践指导。

Deolin昨天晚上阅读了这个项目的源码。

Deolin昨天晚上获得了较大的收获。



## 使用方式

项目主体只有控制层，并集成了Jetty的Maven插件，所以直接运行即可。

```shell
$ mvn jetty:run
```



访问

```http
http://localhost:8080/spring-mvc-showcase
```



## @RequestParam绑定列表对象

~~~http
/demo?values=1&values=2&values=3
~~~



~~~~java
@GetMapping
void get(@RequestParam List<Integer> ) {
    
}
~~~~



## 使用DTO绑定多个@RequestParam参数

~~~http
/demo?v1=&v2=2&v3=&v4=true
~~~



**过去的方式**

~~~java
void get(@RequestParam(defaultValue = "a") String v1,
        @RequestParam @Max(10) Integer v2,
        @RequestParam(required = false) BigDecimal v3,
        @RequestParam Boolean v4
) {
}
~~~



**使控制层更简练的方式**

~~~java
void get(@Valid JavaBean javaBean) {
}
~~~



~~~java
@Data
class JavaBean {

    private String v1;

    @NotNull // 之前没有声明required = false都需要NotNull
    @Max(10)
    private Integer v2;
    
    private BigDecimal v3;
    
    @NotNull
    private Boolean v4;

    public JavaBean() {
        // defaultValue
        v1 = "a";
    }
    
}
~~~



注意这种用法与**绑定请求体**是不同的，对时间类型的格式化注解也是不同的。下面两张图可以直观地反映它们的区别

![](images/spring-mvc-showcase-01.png)



![](images/spring-mvc-showcase-02.png)

