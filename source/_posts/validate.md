---
title: 数据校验

date: 2018-05-23 16:54

updated: 2018-05-23 16:54

tags:
- Spring

categories: Java

permalink: validate
---

## 简介

作为服务端，应该不相信任何客户端传来的数据。这篇POST介绍了基于Spring的Web项目中如何做数据校验。



## 校验注解与校验框架

Java规范提供了很多校验注解，位于`javax`包下，但是这些注解仅仅是一种规范，我们需要具体的实现。

~~~xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.10.Final</version>
</dependency>
~~~



如果是Spring Boot项目，那么`hibernate-validator`依赖也不是必须的，因为`spring-boot-starter-web`内部已经继承了它。



## 哪些东西需要被检验

对于web项目来说，不可信任的数据来源一般只有控制层的形参，一共有三类。

1. 请求参数

   ~~~java
       @GetMapping("/a")
       public String get(@RequestParam String url) {
           return url;
       }
   ~~~

2. 请求体

   ~~~java
       @PostMapping("/b")
       public String post(@RequestBody User user) {
           return user.toString;
       }
   
       @Date
       public static class User { // 考虑到篇幅，这里将DTO写成控制器的静态内部类
           private Long id;
           private String name;
           private Integer age;
       }
   ~~~

3. url片段

   ~~~java
       @GetMapping("/c/{id}")
       public String get2(@PathVariable Long id) {
           return id;
       }
   ~~~



第1种与第3种，区别在于*参数绑定*，对于数据校验而言，他们是同样的情况。

也就是说，两类东西需要被校验——请求方法的**参数对象**和**参数对象内部的属性**。



## 校验参数对象

首先需要控制层声明`@Validated`，以开启对参数对象的校验

~~~java
@RestController
@Validated
public class UserController {

// ...
~~~



然后为具体的参数追加你所需要的校验注解

~~~java
    @GetMapping("/a")
    public String get(@RequestParam @Length(max = 255) String url) {
        return url;
    }
    
    @GetMapping("/c/{id}")
    public String get2(@PathVariable Long id) {
        return id;
    }
~~~



一旦校验未通过，会抛出`javax.validation.ConstraintViolationException`异常，可以在统一异常处理中对其进行捕获，解析，处理。



至此，完毕。可以看到，`@Validated`的作用域是整个控制器，声明`@Validated`的控制器里面每个请求方法的参数对象都会得到校验（前提是声明了具体校验注解）。



## 校验参数对象内部的属性

首先需要在参数对象前声明`@Valid`，以开启对其内部属性的校验

~~~java
    @PostMapping("/b")
    public String post(@RequestBody @Valid User user) {
        return user.toString;
    }
~~~



然后为具体的内部参数追加你所需要的校验注解

~~~java
    @Date
    public static class User {

        private Long id;

        @NotBlank
        @Length(max = 255)
        private String name;

        @NotNull
        @Max(199)
        private Integer age;

    }
~~~



一旦校验未通过，会抛出`org.springframework.web.bind.MethodArgumentNotValidException`异常，可以在统一异常处理中对其进行捕获，解析，处理。



至此，完毕。可以看到，`@Valid`的作用域是具体的参数对象，声明`@Valid`的参数里面每个内部属性都会得到校验（前提是声明了具体校验注解）。



## 分组校验

一个DTO时常可以被复用，如下所示

~~~java
@RestController
public class UserController {
    
    /**
     * 创建用户
     */
    @PostMapping
    public String create(@RequestBody @Valid User user) {
        // ...
    }
    
    /**
     * 修改用户
     */
    @PutMapping
    public String update(@RequestBody @Valid User user) {
        // ...
    }
    
    @Date
    public static class User {

        private Long id;

        @NotBlank
        @Length(max = 255)
        private String name;
        
        // ...
    }
}
~~~



`User`类是两个请求方法的参数类型，很容易想到，`修改用户`时，`id`是不能为空的，而`创建用户`时，`id`是可以为空的，两种校验策略，似乎需要分别定义一个`CreateUser`类和`UpdateUser`类？

定义两个`User`类非常不优雅，因为除了`id`以外，在`修改用户`和`创建用户`时，其他属性的校验策略是完全一样的。

更好的方法是使用校验注解的`group`属性，指定校验组



首先，定义一个校验组标记

~~~java
/**
 * “更新”校验组标记
 */
public interface Update {}
~~~

~~~java
/**
 * “创建”校验组标记
 */
public interface Create {}
~~~



然后，为`id`属性的校验注解指定`group`属性，以声明自己属于什么校验组

~~~java
    @Date
    public static class User {

        @NotNull(group = Update.class)
        private Long id;

        @NotBlank
        @Length(max = 255)
        private String name;
        
        // ...
    }
~~~



最后，将`@Valid`改成`@Validated`，并指定启用的校验组

~~~java
	/**
     * 创建用户
     */
    @PostMapping
    public String create(@RequestBody @Validated(Create.class) User user) {
        // ...
    }
    
    /**
     * 修改用户
     */
    @PutMapping
    public String update(@RequestBody @Validated(Update.class) User user) {
        // ...
    }
~~~



这样一来，`update()`方法会去校验`id`，`create()`方法会无视`id`上声明的`@NotNull`，不去校验。



## 分组校验的劣势

在上面的例子中，由于`User.name`的校验注解没有指定`group`，这些注解不属于任何一个校验组，想要`User.name`的校验即在`创建用户`时，又在`更新用户`时生效，需要指定所有的校验组标记。

~~~java
        @NotBlank(groups = {Create.class, Update.class})
        @Length(max = 255, groups = {Create.class, Update.class})
        private String name;
~~~



太长了。。。。借助接口继承，我们可以让它短一点

~~~java
/**
 * 校验组标记：所有场合
 */
public interface All {}
~~~

~~~java
/**
 * “更新”校验组标记
 */
public interface Update {}
~~~

~~~java
/**
 * “创建”校验组标记
 */
public interface Create {}
~~~

~~~java
        @NotNull(groups = Update.Class)
        private Long id;

        @NotBlank(groups = All.class)
        @Length(max = 255, groups = All.class)
        private String name;
~~~



短了一点，但依然太长，但是暂时没有更好的方法。



## 自定义注解

`hibernate-validator`提供了接口让我们能够自定义校验注解，下面这个示例介绍了如何自定义校验注解。

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
public @interface Mobile{

    String message() default "不是正确的手机号";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

}
~~~

~~~java
/**
 * “手机号”校验器
 *
 * @author Deolin
 */
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



## 使用自定义注解代替分组校验

上面那个分组校验的示例中，需要反复为注解声明`groups = All.class`的原因是，两个请求检验策略的区别仅仅在于**`id`能不能为null**。



那么为什么不专门定义一个自定义注解，通过反射来校验参数对象内部的某个属性不能为空呢？Deolin设计了一个`@Require`注解——

~~~java
/**
 * “必选项”校验用注解
 * <pre>
 * 支持类型：Object
 * 规则：被声明的对象中，指定字段必须不为null。如果不指定{value}，则本注解不会产生任何作用
 * </pre>
 */
@Documented
@Constraint(validatedBy = {RequireValidator.class})
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
@Retention(RUNTIME)
public @interface Require {

    String message() default "必选项为null";

    Class<?>[] groups() default {};

    String[] value();

    Class<? extends Payload>[] payload() default {};

}
~~~

~~~java
/**
 * “必选项”校验器
 *
 * @author Deolin
 */
public class RequireValidator implements ConstraintValidator<Require, Object> {

    private String[] fieldNames;

    @Override
    public void initialize(Require constraintAnnotation) {
        fieldNames = constraintAnnotation.value();
    }

    @Override
    public boolean isValid(Object value, ConstraintValidatorContext context) {
        if (value == null || fieldNames == null || fieldNames.length == 0) {
            return true;
        }
        Class clazz = value.getClass();
        for (String fieldName : fieldNames) {
            Field field = ReflectionUtils.findField(clazz, fieldName);
            if (field == null) {
                throw new RuntimeException("类" + clazz + "不存在字段" + fieldName);
            }
            field.setAccessible(true);
            Object fieldValue = ReflectionUtils.getField(field, value);
            if (fieldValue == null) {
                return false;
            }
        }
        return true;
    }

}
~~~



直接把`@Require`声明在`User`前面，就能对`id`是否为null进行校验了。

~~~java
	/**
     * 创建用户
     */
    @PostMapping
    public String create(@RequestBody @Valid User user) {
        // ...
    }
    
    /**
     * 修改用户
     */
    @PutMapping
    public String update(@RequestBody @Valid @Require("id") User user) {
        // ...
    }
~~~

注意，`@Require`校验的是参数对象，所以生效的前提是控制器声明了`@Validated`。

有了`@Require`以后，一切有关分组校验的属性都不再需要了。



## 校验List对象

如果List对象是参数对象的内部属性，那么只需要递归地声明`@Valid`即可。

~~~java
    /**
     * 批量创建用户
     */
    @PostMapping
    public String batchCreate(@RequestBody @Valid UserList user) {
        // ...
    }

    public static class UserList {

        @Valid
        private List<User> users;
        
    }

    public static class User {
        // ...
    }
~~~



如果`@RequestBody`对象是一个List对象，那么就需要专门定义一个List的派生类了，具体可以参考Deolin的[这篇POST](http://spldeolin.com/posts/valid-list/)。