---
title: 初次使用RestController注解 时发生的Ambiguous mapping问题

date: 2018-01-07 12:17

updated: 2018-01-07 12:17

tags: 启动报错

categories: Java

permalink: ambiguous-mapping-when-use-restcontroller
---

一个因为Deolin记忆混淆而引发的问题。

`@RestController`是`@Controller`和`@ResponseBody`的结合，而没有结合`@RequestMapping`。

也就是说，下面三个控制器的路由是完全一样的。

~~~java
@RestController("student")
public class StudentController () {}
~~~

~~~java
@RestController("teacher")
public class TeacherController () {}
~~~

~~~java
@RestController
@RequestMapping("/")
public class TestController () {}
~~~

例子中`@RestController`的value属性完全不会发挥作用，这一点可能非常容易跟`@GetMapping`、`@PostMapping`这样的注解混淆起来。

如果不小心混淆了，确实很容易因为存在路由完全的一样的请求方法，而引发`Ambiguous Mapping`问题。

