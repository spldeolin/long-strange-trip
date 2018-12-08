---
title: 控制层日志的实践

date: 2018-03-04 09:25:38

updated: 2018-03-04 09:25:38

tags:
- Spring
- AOP

categories: Java

permalink: controller-log
---

#### 简介

借助AOP，对进出控制层的请求进行日志打印，顺便解析了参数和返回值。

#### 切面类

```java
@Aspect
@Component
@Log4j2
public class RequestLogAspect {

    private final ParameterNameDiscoverer parameterNameDiscoverer = new LocalVariableTableParameterNameDiscoverer();

    @Pointcut("execution(* com.spldeolin.restfulapi.controller..*.*(..))")
    public void performance() {}

    @Before("performance()")
    public void before(JoinPoint point) throws NoSuchMethodException {
        HttpServletRequest request = RequestContextUtil.request();
        Method method = ((MethodSignature) point.getSignature()).getMethod();
        String[] paramNames = parameterNameDiscoverer.getParameterNames(method);
        Object[] paramValues = point.getArgs();
        // 标识
        String insignia = RandomStringUtil.generateEnNum(6);
        RequestContextUtil.createInsignia(insignia);
        // 日志
        log.info("");
        log.info("收到请求。标识：" + insignia);
        log.info("URL：" + request.getRequestURI());
        log.info("控制器类名：" + point.getTarget().getClass().getSimpleName());
        log.info("请求方法名：" + method.getName());
        for (int i = 0; i < paramNames.length; i++) {
            log.info("请求方法参数" + paramNames[i] + "：" + paramValues[i]);
        }
        log.info("开始处理...");
    }

    @AfterReturning(value = "performance()", returning = "requestResult")
    public void after(Object requestResult) {
        log.info("...处理完毕");
        log.info("请求方法返回值：" + requestResult);
        log.info("返回响应。标识：" + RequestContextUtil.insignia());
        log.info("");
    }

}
```

```java
// RequestContextUtil.java的代码片段

/**
     * 为当前HTTP请求生成一个标识码。
     *
     * @param insignia 标识码
     */
    public static void createInsignia(String insignia) {
        request().setAttribute("CURRENT_REQUEST_INSIGNIA", insignia);
    }

    /**
     * 取得当前HTTP请求的标识码。
     *
     * @return 标识码
     */
    public static String insignia() {
        return (String) request().getAttribute("CURRENT_REQUEST_INSIGNIA");
    }
```


这里引入了一个“标识”的概念，打印标识的意图是能够在复杂的日志中定位到请求结束的日志位置。

假如，被有个控制层调用的业务层中用大量的日志会打印，或者，多个请求在一段时间内先后被发起，那么仅仅通过“收到请求”“返回响应”这样的汉字，已经很难看清某次请求日志的开始和结束位置了，而有了“标识”，能清晰不少。

> 当然，log4j2的[%thread]片段确实也能把不同的请求区分开，但是从长度上来讲，它可能不是特别得清晰和直观。

除此以外，由于“标识”是设置在request.attribute中的，那么“标识”也能提供给统一异常处理使用。完全可以在统一异常处理被击穿的场合，顺便打印“标识”，开发人员通过“标识”就能直接定位到“收到请求”的位置，从而得知这次异常请求的参数等信息。

#### 演示

- 示例代码1

```java
    @ApiOperation("通过生日和起床时间查找学生")
    @GetMapping("by_birthday_and_daily_warkup_time")
    public RequestResult byBirthdayAndDailyWarkupTime(
            @ApiParam("生日（年月日）") @RequestParam("birthday")
            @DateTimeFormat(pattern = "yyyy-MM-dd") @Past Date birthday,
            @ApiParam("日常起床时间（时分秒）") @RequestParam("daily_warkup_time")
            @DateTimeFormat(pattern = "HH:mm:ss") Date dailyWarkupTime) {
        return RequestResult.success("查找所有birthday为" + birthday + "，daily_warkup_time为" + dailyWarkupTime + "的学生。");
    }
```

- 示例代码2

```java
    @ApiOperation("通过生日和起床时间查找学生")
    @GetMapping("by_birthday_and_daily_warkup_time")
    public RequestResult byBirthdayAndDailyWarkupTime(
            @ApiParam("生日（年月日）") @RequestParam("birthday")
            @DateTimeFormat(pattern = "yyyy-MM-dd") @Past Date birthday,
            @ApiParam("日常起床时间（时分秒）") @RequestParam("daily_warkup_time")
            @DateTimeFormat(pattern = "HH:mm:ss") Date dailyWarkupTime) {
        Integer.valueOf("抛我");
        return RequestResult.success("这里已经不重要了");
    }
```

#### 详细示例

[spldeolin / restful-api](https://github.com/spldeolin/restful-api "spldeolin/restful-api: self discipline...")
