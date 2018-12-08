---
title: 基于Shiro的token过滤器

date: 2018-05-19 14:14:00

tags:
- Shiro

categories: Java

permalink: shiro-token-filter
---

## 简介

有些请求，在生产环境中，一般是提供给运维人员或者开发人员调用，用于获取App运行时的各种实时指标。

这些请求，基本上就是`Actuator`提供的那些接口了，我们需要为这类请求做一些访问控制，而它们专门预留一套权限又显得不够优雅。

这篇POST介绍了基于Shiro的token过滤器，用于对一些非业务的接口进行访问控制。在开始这篇POST之前，需要确保Spring正确地集成了Shiro。



## Token生成器

~~~java
@Component
@Data
@Log4j2
public class TempTokenHolder {

    private String actuatorToken;

    @Scheduled(fixedDelay = 24 * 3600 * 1000)
    public void initTokenWhenAppStartUp() {
        actuatorToken = UUID.randomUUID().toString();
        log.info("actuatorToken");
        log.info(actuatorToken);
    }

}
~~~

每24小时生成一个token，token可以通过日志、短信、邮件或是别的什么方式保存下来。



## Shiro自定义过滤器

~~~java
/**
 * Actuator相关请求过滤器。
 * <pre>
 * 用于处理actuator提供的相关请求，为这类请求专门提供过滤策略
 * </pre>
 */
@AllArgsConstructor
public class ActuatorFilter extends AccessControlFilter {

    public static final String MARK = "actuator";

    private Environment environment;

    private TempTokenHolder tempTokenHolder;

    @Override
    protected boolean isAccessAllowed(ServletRequest req, ServletResponse resp, Object mappedValue) {
        // 非生产环境直接放行
        if (!ArrayUtils.contains(environment.getActiveProfiles(), CoupledConstant.PROD_PROFILE_NAME)) {
            return true;
        }
        // 生产环境需要专门的token
        String token = req.getParameter("token");
        if (tempTokenHolder.getActuatorToken().equals(token)) {
            return true;
        }
        return false;
    }

    // 如果isAccessAllowed()返回false，则调用这个方法
    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
        // 不保存request，直接重定向掉
        WebUtils.issueRedirect(request, response, UrlForwardToExceptionController.UNAUTHORIZED_URL);
        return false;
    }

}
~~~



## 注册过滤器

~~~java
@Configuration
public class ShiroConfig {

    @Value("${management.context-path}")
    private String actuatorUrlPrefix;
    
    @Autowired
    private Environment environment;
    
    @Autowired
    private TempTokenHolder tempTokenHolder;
    
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 过滤器
        shiroFilterFactoryBean.setFilters(createFilters());
        // 过滤器链
        shiroFilterFactoryBean.setFilterChainDefinitionMap(createFilterChainDefinitions());
        return shiroFilterFactoryBean;
    }
    
    /**
     * @return 过滤器一览
     */
    private Map<String, Filter> createFilters() {
        Map<String, Filter> filters = new HashMap<>();
        /* 又需要的话可以追加其他自定义过滤器 */
        filters.put(ActuatorFilter.MARK, new ActuatorFilter(environment, tempTokenHolder));
        return filters;
    }
    
    /**
     * @return 过滤器链
     */
    private Map<String, String> createFilterChainDefinitions() {
        Map<String, String> filterChainDefinitions = new HashMap<>();
        // actuator相关请求使用专门的过滤器
        filterChainDefinitions.put(actuatorUrlPrefix + "/**", ActuatorFilter.MARK);
        
        // 其余的过滤器链省略...
        
        return filterChainDefinitions;
    }
    
    // SecurityManager、Realm、CacheManager等相关配置省略...
    
}
~~~



## 重定向Controller

过滤器`onAccessDenied`方法抛出的异常无法被**统一异常处理**捕获到（因为此时还执行到控制层），所以需要创建一个Controller作为桥梁。

~~~java
/**
 * 控制器：提供URL转发到统一异常处理的映射
 */
@Controller
@RequestMapping
public class UrlForwardToExceptionController {
   
    public static final String UNAUTHORIZED_URL = "/unauth";
    
    @RequestMapping(UNAUTHORIZED_URL)
    public void unauth() {
        throw new AuthorizationException();
    }
    
}
~~~



## 统一异常处理

无法通过过滤的请求最终交由**统一异常处理**来处理。

~~~java
    /**
     * 403 没权限或是请求actuator时提供的token不正确
     */
    @ExceptionHandler(AuthorizationException.class)
    public RequestResult handleAuthorizationException() {
        return RequestResult.failure(ResultCode.FORBIDDEN);
    }
~~~



## 效果

首先，使用`dev`环境启动项目——

![](images/shiro-token-filter-1.png)



访问`/actuator/beans`

![](images/shiro-token-filter-2.png)



没有任何问题。

然后使用`prod`环境启动项目——

![](images/shiro-token-filter-3.png)



记录一下token

![](images/shiro-token-filter-4.png)



不带token直接请求`/actuator/beans`

![](images/shiro-token-filter-5.gif)



带正确的token请求`/actuator/beans`

![](images/shiro-token-filter-6.png)



跟预想一致。



## 后记

至此，token过滤器设计完毕，上面的代码片段提供了大致思路，源码可以参照[spldeolin](https://github.com/spldeolin) / [beginning-mind](https://github.com/spldeolin/beginning-mind)