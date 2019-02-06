---
title: Spring Boot集成Shiro

date: 2018-04-22 18:17

updated: 2018-04-22 18:17

tags:
- Spring Boot
- Shiro

categories: Java

permalink: springboot-shiro
---

## 简介

这篇POST介绍了Spring Boot集成Shiro。

为了使项目能向分布式伸展，所以直接把**分布式缓存**和**分布式会话**考虑进去。

- 分布式缓存通过Spring Session直接就能实现
- 分布式会话需要借助Redis，重写Shiro的缓存管理策略（`CacheManager`）

我们假定项目事先集成Spring Session，如果情况不是如此，请先去集成Spring Session。



## 追加依赖

~~~xml
<!--shiro-->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
<!--shiro redis-->
<dependency>
    <groupId>org.crazycake</groupId>
    <artifactId>shiro-redis</artifactId>
    <version>3.0.0</version>
</dependency>
~~~



## 编写自定义组件

### Realm

~~~java
/**
 * 对接业务模块的自定义Shiro Realm 
 *
 * @author Deolin
 */
@Component
public class ServiceRealm extends AuthorizingRealm {

    /**
     * “帐号”表的业务
     */
    @Autowired
    private SecurityAccountService securityAccountService;

    /**
     * 认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(
            AuthenticationToken authenticationToken) throws AuthenticationException {
		// authenticationToken是控制层调用subject.login(AuthenticationToken token)传入的对象，
        // 即用户输入的登录信息。
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        String username = token.getUsername();
        // 通过登录信息查找DB中的登录数据（这部分需要根据实际表结构设计，Deolin给出了一种实践作为示例）
        SecurityAccount securityAccount = securityAccountService.searchOne("username",
        		username).orElseThrow(() -> new UnknownAccountException("用户不存在或密码错误"));
        if (!securityAccount.getEnableSign()) {
            throw new DisabledAccountException("用户已被禁用");
        }
        // 将登录者的信息和登录信息等封装到一个领域对象，
        // 保存到AuthorizationInfo中，作为缓存受Shiro管理
        CurrentSigner currentSigner = CurrentSigner.builder().sessionId(
                RequestContextUtil.session().getId()).securityAccount(securityAccount).signedAt(
                LocalDateTime.now()).build();
        return new SimpleAuthenticationInfo(currentSigner, token.getPassowrd, getName());
    }

    /**
     * 认证后授权
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        Long accountId = ((CurrentSigner) principals.getPrimaryPrincipal()).getSecurityAccount().getId();
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        // 通过“帐号”查找“帐号”所拥有的权限（这部分需要根据实际表结构设计，Deolin给出了一种实践作为示例）
        List<String> permissionMappings = securityAccountService.listAccountPermissionMappings(accountId);
    	Set<String> permissionNames = Sets.newHashSet(permissionMappings);
        info.setStringPermissions(permissionNames);
        return info;
    }

}
~~~



### CredentialsMatcher

~~~java
@Component
public class TinyCredentialsMatcher extends SimpleCredentialsMatcher {

    @Override
    public boolean doCredentialsMatch(AuthenticationToken authenticationToken, 
            AuthenticationInfo info) {
        // info来自数据库，token来自用户输入
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        String realPassword = new String(token.getPassword());
        String tokenPassword = (String) info.getCredentials();
        if (!super.equals(realPassword, tokenPassword)) {
            throw new IncorrectCredentialsException("用户不存在或密码错误");
        }
        return true;
    }

}
~~~



## CurrentSigner

~~~java
/**
 * 当前登录者的信息
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Accessors(chain = true)
public class CurrentSigner implements Serializable, AuthCachePrincipal {

    /**
     * 会话ID
     */
    private String sessionId;

    /**
     * 帐号
     */
    private SecurityAccount securityAccount;

    /**
     * 登录时间
     */
    private LocalDateTime signedAt;

    private static final long serialVersionUID = 1L;

    @Override
    public String getAuthCacheKey() {
        return securityAccount.getUsername();
    }

}
~~~

这里必须实现`AuthCachePrincipal`接口，否则后面调用`subject.logout()`将会报错。



## 注册组件

找一个被`@Configuration`声明的组件（或是新建一个），追加以下Bean的配置

### 过滤器

~~~java
    @Bean
    public ShiroFilterFactoryBean shiroFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 重定向到一个RestController
        shiroFilterFactoryBean.setLoginUrl("/unauthc");
        // 放行登录请求、Spring Boot的error页面
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        filterChainDefinitionMap.put("/security/sign_in", "anon");
        filterChainDefinitionMap.put("/error", "anon");
        // 也可以根据需要，在某些activeProfile放行一个特殊的请求（如swagger、actuator等）
        // ...
        filterChainDefinitionMap.put("/**", "authc");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }
~~~



### SecurityManager

~~~java
    @Bean
    public SecurityManager securityManager(AuthorizingRealm realm, CacheManager cacheManager) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(realm);
        securityManager.setCacheManager(cacheManager);
        return securityManager;
    }
~~~



### Realm和CredentialsMatcher

~~~java
    @Bean
    public AuthorizingRealm realm(SimpleCredentialsMatcher simpleCredentialsMatcher,
            CacheManager cacheManager) {
        AuthorizingRealm realm = new ServiceRealm();
        realm.setCredentialsMatcher(new TinyCredentialsMatcher());
        realm.setCacheManager(cacheManager);
        return realm;
    }
~~~



### CacheManager和RedisManager

这两个组件由`shiro-redis`依赖提供。

注册`RedisManager`需要用到`Redis`在`application.yml`中的相关配置。

~~~java
    @Bean
    public RedisCacheManager cacheManager(RedisManager redisManager) {
        RedisCacheManager redisCacheManager = new RedisCacheManager();
        redisCacheManager.setRedisManager(redisManager);
        return redisCacheManager;
    }

    @Bean
    public RedisManager redisManager(
            @Value("${spring.redis.host}") String host,
            @Value("${spring.redis.port}") int port,
            @Value("${spring.redis.timeout}") int timeout,
            @Value("${spring.redis.password}") String password) {
        RedisManager redisManager = new RedisManager();
        redisManager.setHost(host);
        redisManager.setPort(port);
        redisManager.setTimeout(timeout);
        redisManager.setPassword(password);
        return redisManager;
    }
~~~



### 开启@RequiresPermissions注解的配置

~~~java
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }
~~~



## Restful“重定向”控制器

~~~java
@RestController
public class RedirectController {

    @GetMapping("unauthc")
    public RequestResult unauthc() {
        return RequestResult.failture(401, "未登录或登录超时");
    }

}
~~~

这里可以根据实际项目的前后端交互规约，自由设计。



## 总结

至此，集成Shiro初步完成。

可以看到，Spring Boot集成Shiro会集成其他的功能麻烦一点点，不过依然比Spring/SpringMVC集成Shiro方便很多（收益于Spring Session和那个第三方的shiro-redis）。



这篇POST只介绍了如何集成Shiro，很多实现方式仅仅只是示例，一定不是最优的，更好的实践方式Deolin将会在后续的POST中介绍，请期待。