---
title: Spring Boot集成Spring Session

date: 2018-04-21 18:35:00

tags:
- Spring Boot
- Spring Session
- Redis

categories: Java

permalink: spring-boot-spring-session
---

## 简介

这篇POST介绍了Spring Boot集成了Redis之后，集成Spring Session，使整个项目实现**分布式会话**的效果。

集成之前首先要确保[Redis](http://spldeolin.com/posts/spring-boot-redis/)被正确集成。



## pom追加依赖

~~~xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
    <version>1.3.2.RELEASE</version>
</dependency>
~~~



## 启动Redis Http Session

在任意一个注册到Spring容器的Bean上面声明`@EnableRedisHttpSession`即可。

不声明的话，启动会报错。

至此，集成完毕。



## 测试

编写一个测试用`controller`

~~~java
@RestController
@RequestMapping("ses")
public class SessionTestController {
    
    @GetMapping("set")
    public String setSes(HttpSession ses) {
        ses.setAttribute("one-cookie", "会话中的曲奇饼干");
        return "SUCCESS";
    }

    @GetMapping("get")
    public String getSes(HttpSession ses) {
        return (String) ses.getAttribute("one-cookie");
    }

}
~~~

首先启动项目，请求`/ses/set`——

![](/images/spring-boot-spring-session-1.png)

请求成功，并且通过`RedisDesktopManager`，可以看到会话已经存储到了`Redis`——

![](/images/spring-boot-spring-session-2.png)

会话在`Redis`中的失效时间由`@EnableRedisHttpSession`注解的`maxInactiveIntervalInSeconds`属性决定，默认为`1800秒`



然后，停止项目后重启，请求`/ses/get`——

![](/images/spring-boot-spring-session-3.png)

请求成功，并且与预想的结果完全一致。



也就是说，只要会话在保持在`Redis`中，那么，同一个浏览器的请求无论被负载均衡到哪个应用服务器，都能借助浏览器的`jsessionid`，从`Redis`中获取正确的会话对象。

至此，分布式会话的效果已经达成了。