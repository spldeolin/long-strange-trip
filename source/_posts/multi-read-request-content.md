---
title: 调用getInputStream()后发生Stream closed的解决方法

date: 2018-12-04 18:22

updated: 2018-12-04 18:22

tags:
- 报错 & BUG

categories: Java

permalink: multi-read-request-content
---



## 简介

实际开发中，我们往往希望能够将每次POST请求的content作为日志保存下来。

你能搜到这篇POST，十有八九也和Deolin一样，想要在过滤器或是Spring拦截器中获取POST请求content中的数据。但是，一旦这样做，Web服务收到请求就会抛出`Stream closed`的异常。

这篇POST将会演示如何解决这个问题。



## 引起异常的原因

主流的Servlet容器（Tomcat、Undertow等）**只允许读取一次**请求的content。

content需要通过调用`request.getInputStream()`来获取，SpringMVC需要获取content，并转化为对象，绑定到Controller中的`@RequestBody`参数上。

如果开发者抢先在过滤器或是拦截器中主动调用`request.getInputStream()`以获取content，那么轮到SpringMVC调用的时候，会直接抛出异常`Stream closed`，因为这个Input流只允许读取一次。



## 演示

~~~java
@Component
@Log4j2
public class GlobalFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        ContentCachingRequestWrapper wrappedRequest = new ContentCachingRequestWrapper(request);
           
        filterChain.doFilter(wrappedRequest, response);
           
        log.info(new String(wrappedRequest.getContentAsByteArray(), wrappedRequest.getCharacterEncoding()));
       
    }    
}
~~~



大致思路是把原来的request对象给包装一下，把`getInputStream`方法给修饰一下。

`org.springframework.web.util.ContentCachingRequestWrapper`是包装器类，实现了`HttpServletRequest`接口，也就是说，它能够作为`filterChain.doFilter()`方法的第一个参数。

`ContentCachingRequestWrapper`的构造方法把原来的request对象装配到自己的私有域上，当Spring框架调用`getInputSteam`时，`getInputStream`的修饰方法会顺便把流缓保存下来（`ByteArrayOutputStream`）。之后调用`getContentAsByteArray()`时候其实获取的是这个私有缓存。



## 其他

与`ContentCachingRequestWrapper`相对应的，还有个`ContentCachingResponseWrapper`类，它是用来包装Response对象的，是用于二次读取Response的content的。不过，它还需要多调用一步

~~~java
wrappedResponse.copyBodyToResponse();
~~~

否则Response报文将会没有body。