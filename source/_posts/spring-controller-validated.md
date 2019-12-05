---
title: Spring WebMVC 控制层参数校验

date: 2018-05-23 16:54

updated: 2019-06-24 14:29

tags:
- Spring

categories: Java

permalink: spring-controller-validated
---

## 简介

作为服务端，不应该相信任何客户端传来的数据。

借助Spring Web提供的功能，可以通过简单的编码，实现在Controller层拦截掉很多非法参数值的效果。



## 引入依赖

如果是Spring Boot项目，那么你可以不引入任何额外的依赖，因为`spring-boot-starter-web`内部已经集成了。



如果是Spring / Spring MVC整合的项目，那么需要引入Hibernate Validator

~~~xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.10.Final</version>
</dependency>
~~~



## 对RequestParam方式绑定的参数进行校验

~~~java
@RestController
@Validated
public class ParamValidDemo {

    @GetMapping("/param")
    void param(@RequestParam @Min(0) Integer userAge) {
        // ...
    }
}
~~~

`@Validated`必须声明在那个位置，否则`@Min(0)`不生效



## 对PathVariable方式绑定的参数进行校验

~~~java
@RestController
@Validated // 这个注解是必须的，代表启用对Long userId的校验
public class PathValidDemo {
  
    @GetMapping("path/{userId}")
    void path(@PathVariable @Max(10) Long userId) {
        // ...
    }
}
~~~

`@Validated`必须声明在那个位置，否则`@Max(10)`不生效




## 对请求体JSON内的字段进行校验

~~~java
@RestController
public class JsonValidDemo {

    @PostMapping("/jsonObj")
    void jsonObj(@RequestBody @Valid UserDTO user) {
        // ...
    }
}
~~~

~~~java
@Data
public class UserDTO implements Serializable {
    @Size(max = 2)
    private String username;
    private static final long serialVersionUID = 1L;
}
~~~

`@Valid`必须声明，否则`UserDTO`中的`@Size(max = 2)`不生效



## 对请求体列表类型JSON内的字段进行校验

~~~java
@RestController
@Validated
public class JsonListValidDemo {


    @PostMapping("/jsonList")
    void jsonList(@RequestBody @Valid List<UserDTO> users) {
        // ...
    }
}
~~~

`@Validate`和`@Valid`必须声明，否则`UserDTO`中的`@Size(max = 2)`不生效

特别注意，他是个`List`，对应的是类似于这样的JSON `[ { "username": "Deolin" } ]`



## 对嵌套型的JSON每一层的字段进行校验

~~~java
@RestController
public class JsonValidDemo {

    @PostMapping("/jsonObj")
    void jsonObj(@RequestBody @Valid OrderDTO orderDTO) {
        // ...
    }
}
~~~

~~~java
@Data
public class OrderDTO implements Serializable {

    /**
     * 下单用户
     */
    @Valid
    private UserDTO user;
    /**
     * 商品
     */
    @Valid
    private List<ItemDTO> items;
    
    private static final long serialVersionUID = 1L;
}
~~~

~~~java
@Data
public class ItemDTO implements Serializable {
    @Min(0)
    private Long itemId;
    private static final long serialVersionUID = 1L;
}
~~~

`OrderDTO`中的`@Valid`必须声明，否则`UserDTO`中的`@Size(max = 2)`和`ItemDTO`中的`@Min(0)`不生效

另外，这种情况下虽然有`List<ItemDTO>`，但`@Validated`是不需要的



## 使用自定义注解

~~~java
/**
 * “手机号”校验注解
 * <pre>
 * 支持类型：String
 * 规则：11位数字，以1开头
 * </pre>
 */
@Documented
@Constraint(validatedBy = {MobileValidator.class})
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
public @interface Mobile {
    String message() default "不是正确的手机号";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
~~~

~~~java
public class MobileValidator implements ConstraintValidator<Mobile, String> {
    @Override
    public void initialize(Mobile constraintAnnotation) {
    }
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }
        // 11位数字，以1开头
        return value.length() == 11 && value.startsWith("1");
    }
}
~~~

~~~java
@RestController
@Validated
public class CustomValidDemo {


    @GetMapping("/custom")
    void param(@RequestParam @Mobile String mobiePhone) {
        // ...
    }
}
~~~



## 源码

[spldeolin](https://github.com/spldeolin) / [valid-demo](https://github.com/spldeolin/valid-demo)

