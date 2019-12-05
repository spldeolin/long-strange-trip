---

title: Spring 组件参数校验

date: 2019-12-05 08:16

updated: 2019-12-05 10:20

tags:
- Spring

categories: Java

permalink: spring-component-validated

---

## 简介

这篇POST将会介绍如何为Spring组件中的方法，进行参数校验。

这里“组件”，不包含“控制器”，如果想要了解控制层参数校验，可以参考[这篇POST](https://spldeolin.com/posts/spring-controller-validated/)



## 示例一

- 首先会有一个Service

  ~~~java
  //~ UserApi.java
  public interface UserApi {
      void doSomething(@NotNull @Valid UserParam user, @Min(1) Long userId);
  }
  
  //~ UserApiImpl.java
  @Validated
  @Service
  public class UserApiImpl {
      @Override public void doSomething(UserParam user, Long userId) {
          // ...
      }
  }
  
  //~ UserParam.java
  @Data @Accessors(fluent = true)
  public class UserParam {
                             private Long id;
      @NotBlank @Size(max=4) private String userName;
      @Past                  private LocalDateTime createdAt;
  }
  ~~~

  解释一下

  接口的方法声明指定了所有的校验策略，`@NotNull`、`@Valid`、`@NotBlank`等等

  实现的类决定了校验实现，即通过`@Validated`注解

  > 想象一下，这些校验如果在方法实现中，通过if-throw来实现，一旦field较多，会有多么的麻烦。



- 示例

  ~~~java
  //~ Client.java
  @Component @Log4j2
  public class Client {
      @Autowired private UserApi userApi;
      public void callApi() {
          try {
          	  userApi.doSomething(null, null);
          } catch (ConstraintViolationException case1) {
            	// ...解析case1对象省略
              log.error("实际参数未通过Api的参数校验规则");
          }
        
          try {
              UserParam user = new UserParam()
                  .userName("abcde")
                  .createdAt(LocalDateTime.now().plusSeconds(1));
              userApi.doSomething(user, -2333L);
          } catch (ConstraintViolationException case2) {
              log.error("实际参数未通过Api的参数校验规则");
          }
      }
  }
  ~~~

  当调用方忽视校验直接调用时，动态代理类会直接抛出`ConstraintViolationException`

  宏观上看，这个异常是由`UserApiImpl`抛出的，所以``UserApiImpl``的方法的参数安全得到了保证



- 设计切面

  更进一步地，可以为所有`@Validated`组件设计一个切面，通过日志专门报告那些没有遵循Api参数校验规则的调用行为，提高定位BUG的速度

  这里给出这个切面的大致思路——

  ~~~java
  //~ MethodCallValidatedAspect.java
  @Component @Aspect @Log4j2
  public class MethodCallValidatedAspect {
      /**
       * 声明了@Validated的组件，中的所有方法
       * 排除所有控制器，原因是控制器有专门的ControllerAdvice来处理异常，不需要这个切面
       */
      @Pointcut("@within(org.springframework.validation.annotation.Validated)"
              + "&& !@within(org.springframework.web.bind.annotation.RestController)"
              + "&& !@within(org.springframework.stereotype.Controller)")
      public void validatedComponent() {}
    
      @AfterThrowing(value = "validatedComponent()", throwing = "ex")
      public void logError(JoinPoint point, ConstraintViolationException ex) {
          Object[] args = point.getArgs();
          // ...解析切点point、异常对象ex 省略
          throw ex;
      }
  }
  ~~~



- 效果

  ![](/images/spring-component-validated-01.png)

  可以看到，无论是实际参数的值和错误原因，都能解析得到



## 示例二

示例一演示的是**接口+实现**形式的组件，如果是单个类形式的组件，也是同理，同样需要`@Validated`

~~~java
//~ UserTemplate.java
@Component
@Validated
public class UserTemplate {
    int saveUsers(@NotEmpty @Valid Collection<User> users) {
        // ...
        return 0;
    }
}
~~~



## 示例三

对于一个Mybatis的Mapper接口

虽然没有实现类，但在接口上声明@Validated也能进行参数校验

猜测原因是Mybatis为接口做动态代理时，代理类继承了接口上的@Validated的注解，导致参数校验能够生效