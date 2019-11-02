---
title: Aspect注解并不属于Component的一种

date: 2018-01-07 15:38

updated: 2018-01-07 15:38

tags:
- Spring

categories: Java

permalink: aspect-is-not-a-component
---

...标题很好的表达了这篇文章的中心思想。

事情的起因是Deolin写了一个切面类，由于惯性思维，把`@Aspect`当作`@Controller`或`@ControllerAdvice`在用了，以至于Spring容器没有将该切面类注册进去。

正确的切面类，类级注解应该同时具有`@Aspect`和`@Component`

~~~java
@Component
@Aspect
public class cookieAspect {
    
	@Pointcut("execution(* *(..))"）
    private void point() {}

    @Before("point")
    public Object eatCookie() {
        System.out.println("调用前吃了一个曲奇饼干");
    }
    
}
~~~



