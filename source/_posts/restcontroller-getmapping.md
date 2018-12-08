---
title: RestController注解和GetMapping注解

date: 2018-01-04 21:58:00

updated: 2018-01-04 21:58:00

tags:
- Spring
- Spring Boot

categories: Java

permalink: restcontroller-getmapping
---

## @RestController

`@RestController`可以代替`@Controller`使用，使用了`@RestController`的控制器默认所有请求方法都用了`@ResponseBody`注解。



## @GetMapping

`@GetMapping("oneName")`与`@RequestMapping(value = "oneName", method = RequestMethod.GET)`等价，`@PostMapping`和`@PutMapping`等，同理。



为了使代码简洁一点，优先使用`@RestController`和`@GetMapping`。



## 特别注意

一个Controller中如果有**转发或重定向**方法，则不能用`@RestController`修饰，因为转发或重定向方法不能被`@ResponseBody`修饰；

如果有那种操作HttpServletResponse实现下载功能的方法，则无所谓。