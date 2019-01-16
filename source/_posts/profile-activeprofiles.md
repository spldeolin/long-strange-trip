---
title: Spring Boot中@ActiveProfiles和@Profile的区别

date: 2018-05-17 21:43

updated: 2018-05-17 21:43

tags:
- Spring Boot

categories: Java

permalink: profile-activeprofiles
---



## @ActiveProfiles

` @ActiveProfiles`是Spring Boot的Test starter提供的注解，如果项目是像下面的方式依赖Test starter的话——

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
~~~

那么`@ActiveProfiles`只能在`/src/test/java`中使用，这种方式也是`SPRING INITIALIZR`提供的方式，可以认为时官方的建议方式。



`@ActiveProfiles`一般声明在UT测试类上，用于指定这个测试类里的测试方式运行时的`profiles`，如下所示——

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("dev")
@Log4j2
public class Tests {

	@Autowired
	private String someService;
    
    @Test
    public void test() {
        someService.doSomething();
    }

}
~~~



## @Profile

` @Profile`是Spring Boot核心模块提供的注解，可以声明在项目能扫描到的任何类和其中的方法上。

被`@Profile`声明的类，必须提供满足`value`属性中指定的条件，才会被注册到Spring容器中。参考下面的例子——

~~~java
@Profile("!dev")
@RestController
@RequestMapping("/test")
public class TestController {
    
    @GetMapping("/greet")
    public String greet() {
        return "Hey there!";
    }

}
~~~

在`dev`的环境下，`TestController`将不会是一个`RestController`，直接请求`/test/greet`会返回404。



被`@Profile`和`@Bean`同时声明的方法，必须满足`@Profile`的`value`属性中指定的条件，才会将返回值对象注册到Spring容器中。参考下面的例子——

~~~java
@Configuration
public class someConfig {
    
    @Profile("!dev")
    @Bean
    public Goods goods() {
        Goods goods = new Goods("夜之子小食");
        return goods;
    }

}
~~~

~~~java
@RestController
public class someController {

    @Autowired
    private Goods goods;

    @GetMapping("goodsName")
    public String getGoodsName() {
        return goods.getName();
    }

}
~~~

在`dev`的环境下，`!dev`表达式无法得到满足，那么`someConfig#goods`的返回值，不会注册到Spring容器中，那么项目启动会直接报错，错误内容大致会是

~~~

~~~



## 后记

Deolin估计`@Profile`注解还会有其他的作用，如果Deolin发现了它们，那么这篇POST将会得到更新。