---
title: 组合注解

date: 2019-02-10 08:35

updated: 2019-02-10 08:35

tags:

categories: Java

permalink: concat-annotation
---



## 简介

这篇POST将会介绍一个关于注解的特性。



## 原效果

现在，有一个典型的RESTful控制器

~~~java
@RestController
@RequestMapping("/demo")
public class DemoController {

    @PostMapping("/doSometing")
    public Long doSomething() {
        return 1L;        
    }

}
~~~



可以看到，需要类级注解除了`@RestController`以外，还需要`@RequestMapping`。



## 定义组合注解

可以通过自定义一个注解，将`@RestController`和`@RequestMapping`组合起来。

~~~java
/**
 * @RestController与@RequestMapping的组合注解
 * 
 * @author Deolin 2019-02-08
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RestController
@RequestMapping
public @interface RestMapping {

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

}
~~~



## 使用组合注解

~~~java
@RestMapping("/demo")
public class DemoController {

    @PostMapping("/doSometing")
    public Long doSomething() {
        return 1L;        
    }

}
~~~



## 切面表达式

虽然`DemoController`不再使用`@RestController`，而是使用了`@RestMapping`，但是`DemoController`**依然可以作为基于@RestController注解的切点**，如下所示

~~~java
@Component
@Aspect
@Log4j2
public class ControllerAspect {

    // 这个切点依然可以切到DemoController中
    @Pointcut("@within(org.springframework.web.bind.annotation.RestController)")
    public void controllerMethod() {
    }

    @Before("controllerMethod()")
    public void before() {
        log.info("hi there.");
    }

}
~~~



