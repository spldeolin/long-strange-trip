---
title: 在Spring中处理HTTP404错误

date: 2018-05-11 17:33:00

tags:
- Spring

categories: Java

permalink: spring-custom-404
---

## 简介

对于一个Spring Boot Web项目而言，当浏览器访问一个不存在的接口时，会得到一个`Whitelabel Error Page`页面，Spring / SpringMVC项目中也有类似的情况，这篇POST介绍了处理HTTP404错误，并返回一个自定义的数据结构的若干种方法。

但是统一异常处理始终对`HTTP 404`这类问题没有任何办法，这篇POST将会介绍一下如何定制HTTP 404的返回数据结构。



## 控制器

~~~java
/**
 * 控制器：单纯返回错误码
 * <pre>
 * 这个控制器往往用于被转发，或是被重定向
 * </pre>
 *
 * @author Deolin
 */
@RestController
@RequestMapping("/")
@Log4j2
public class FailureController {
    
    public static final String NOT_FOUND_MAPPING = "/404";

    public static final String NOT_FOUND_PARAM_METHOD = "method";

    public static final String NOT_FOUND_PARAM_URL = "url";

    @RequestMapping(NOT_FOUND_MAPPING)
    public RequestResult notFound(@RequestParam(value = NOT_FOUND_PARAM_METHOD, required = false) String method,
            @RequestParam(value = NOT_FOUND_PARAM_URL, required = false) String url) {
        if (StringUtils.isNoneEmpty(method, url)) {
            return RequestResult.failure(ResultCode.NOT_FOUND, "请求不存在。[" + getFirstPart(method) + "]" + getFirstPart(url));
        } else {
            return RequestResult.failure(ResultCode.NOT_FOUND);
        }
    }

    private String getFirstPart(String s) {
        return StringUtils.split(s, ',')[0];
    }
    
}
~~~

一切HTTP404的请求都应该转发到这个接口。



## 修饰内置Servlet容器

追加对内置Servlet容器的配置

~~~java
/**
 * 内置Servlet容器配置
 *
 * @author Deolin
 */
@Configuration
public class EmbeddedServletContainerConfig implements EmbeddedServletContainerCustomizer {

    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        container.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, FailureController.NOT_FOUND_MAPPING));
    }

}
~~~



这个方法十分简洁优雅，在绝大多数场合没有任何问题，它甚至把静态资源的404都处理了（...如果在`WebMvcConfigurerAdapter`配置了静态资源映射的话）



如果项目将来不是通过内置容器运行的，那HTTP404更不是问题了，直接在外置容器配置一下404页面即可。



## 自定义过滤器

假如项目集成了`Shiro`作为安全框架，并且为了确保**接口的闭环性**，在Shiro的过滤器链最后追加了一条`/**=authc`，那么，在未登录的情况下，访问本应该返回`请求不存在`的请求，会变成返回`未登录`。



因为任何不存在的请求的URL，都是属于`/**`的范畴，这些请求会先进入Shiro的过滤器，过滤器发现你没有登录，会将请求直接重定向到对Shiro而言的“登录页面”，即便这个请求确实是不存在，也没机会进入到统一异常处理那一层了。



想要达到以下的需求，那么还需要定义一个过滤器。

​	*无论用户登录与否，访问不存在的请求都应该返回”请求不存在。“这样的信息。*



我们采用一种**过滤+转发**的思路来设计，首先，定义一个过滤器，并确保它在Shiro的过滤器之前被调用到。

~~~java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE) // 确保在Shiro过滤器之间被调用
public class NotFoundFilter implements Filter {

    @Autowired
    private RequestMappingHandlerMapping requestMappingHandlerMapping;

    @Override
    public void doFilter(ServletRequest req, ServletResponse res,
            FilterChain chain) throws IOException, ServletException {
    }

    @Override
    public void init(FilterConfig config) {
    }

    @Override
    public void destroy() {
    }

}
~~~



接着，注入`RequestMappingHandlerMapping`（真是个猎奇的类名），通过这个对象来获取容器的所有请求的Mapping。

~~~java
    @Autowired
    private RequestMappingHandlerMapping requestMappingHandlerMapping;
~~~



然后，补全`doFilter()`方法，主要逻辑是——如果进入的请求匹配不到Mapping，那么将请求转发到一个返回错误码的控制器；如果匹配得到，那就放行。

~~~java
    @Override
    public void doFilter(ServletRequest req, ServletResponse res,
            FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        String requestUrl = request.getRequestURI();
        // 放行静态资源请求
        if (requestUrl.startsWith(properties.getFile().getMapping())) {
            chain.doFilter(req, res);
            return;
        }
        // 匹配
        boolean findable = false;
        for (Map.Entry<RequestMappingInfo, HandlerMethod> entry : requestMappingHandlerMapping.getHandlerMethods()
                .entrySet()) {
            RequestMappingInfo info = entry.getKey();
            if (null != info.getMatchingCondition((HttpServletRequest) req)) {
                findable = true;
            }
        }
        // 匹配不到
        if (!findable) {
            // 转发到FailureController
            String forwardUrl = FailureController.NOT_FOUND_MAPPING +
                    "?" + FailureController.NOT_FOUND_PARAM_METHOD + "=" + request.getMethod() +
                    "&" + FailureController.NOT_FOUND_PARAM_URL + "=" + requestUrl;

            req.getRequestDispatcher(forwardUrl).forward(req, res);
        } else {
            // 放行匹配到的请求
            chain.doFilter(req, res);
        }
    }
~~~



最后，看一下效果。

![](/images/spring-custom-404-1.png)



清干净Cookie，模拟登录超时，然后在未登录的情况下，再看一下效果。

![](/images/spring-custom-404-2.png)



可以看到，完美的达到了上述的需求。



## 其他说明

1. 如果没有特殊的需求，Spring Boot修饰内置Servlet容器，Spring / SpringMVC修改web.xml完全可以满足要求。
2. 示例代码中提到一些类，如`RequestResult`和`ResultCode`没有给出具体的实现，可以参照[splendid](https://github.com/spldeolin) / [beginning-mind](https://github.com/spldeolin/beginning-mind)