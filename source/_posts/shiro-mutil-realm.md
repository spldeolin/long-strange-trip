---
title: Shiro 配置多Realm

date: 2018-08-21 16:55:00

tags:
- Shiro

categories: Java

permalink: shiro-mutil-realm
---



## 简介

这篇POST通过一个实际的需求，介绍如何在Shiro框架下，配置多个Realm。



## 源流

一开始，整个应用只有一个PC端管理后台，之后，需要在追加一个微信端H5版本的管理后台。

微信端除了通过“用户名+密码”登录，还支持通过Openid自动登录，

那么，原本的通过`subject.login(token)`登录的方式就行不通了，

因为密码在DB中是加盐加密后存储的，无法通过Openid查询到用户的明文密码，也就无法构建`token`对象。



## 解决方式

原项目只有一个自定义`Realm`，内部的认证方法是通过用户名查找该用户的密文密码和盐，判断用户输入的密码是否正确。

那么，完全可以再追加一个自定义`Realm`，只要通过用户名查找到的用户的openid与传入的openid一致，就认为认证通过。



## 实现

代码片段如下

~~~java
@Bean
public SecurityManager securityManager(@Autowired @Qualifier("realForCms") AuthorizingRealm realmForCms,
                                       @Autowired @Qualifier("realmForWechatAuto") AuthorizingRealm realmForWechatAuto,
                                       CacheManager cacheManager) {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    // 多REALM认证策略
    securityManager.setAuthenticator(modularRealmAuthenticator());
    // 多REALM
    List<Realm> realms = Lists.newArrayList();
    realms.add(realmForWechatAuto);
    realms.add(realmForCms);
    securityManager.setRealms(realms);
    // 缓存管理
    securityManager.setCacheManager(cacheManager);
    return securityManager;
}


/**
 * 多Realm认证策略
 */
public ModularRealmAuthenticator modularRealmAuthenticator() {
    ModularRealmAuthenticator modularRealmAuthenticator = new ModularRealmAuthenticator();
    // 所有realm中，有一个成功则算成功
    modularRealmAuthenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
    return modularRealmAuthenticator;
}

/**
 * 原Realm
 */
@Bean(name = "realForCms")
public AuthorizingRealm realmForCms(CacheManager cacheManager) {
    AuthorizingRealm realm = new ServiceRealmForCms();
    realm.setCredentialsMatcher(new SaltSha512CredentialsMatcher());
    realm.setCacheManager(cacheManager);
    return realm;
}

/**
 * 追加的新Realm
 */
@Bean(name = "realmForWechatAuto")
public AuthorizingRealm realmForWechatAuto(CacheManager cacheManager) {
    AuthorizingRealm realm = new ServiceRealmForWechatAuto();
    // doCredentialsMatch方法直接返回true的自定义CredentialsMatcher
    realm.setCredentialsMatcher(new AlwaysMatchedCredentialsMatcher());
    realm.setCacheManager(cacheManager);
    return realm;
}
~~~



~~~java
@Log4j2
public class ServiceRealmForWechatAuto extends AuthorizingRealm {

    // 注入用户CRUD的Service

    /**
     * 认证后授权
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        // 省略
    }
    
    /**
     * 认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken)
            throws AuthenticationException {

        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        String principal = token.getUsername();
        User user = userCrudService.searchOne(User.builder().mobile(principal).build()).orElseThrow(
                () -> new UnknownAccountException("用户不存在或是已被删除"));
        if (!user.getEnableSign()) {
            throw new DisabledAccountException("用户已被禁用");
        }

        // 这里的password是微信的openid
        // 通过“用户名密码”方式登录时，这里应该会抛出异常，用户的明文密码与openid一致的情况相当罕见，实际上，注册时限制一下密码的长度，就能杜绝这种情况。
        if (!new String(token.getPassword()).equals(user.getWechatOpenid())) {

            // “微信自动登录”时，这里不应该会抛出异常，除非在高并发情况下，DB中用户的openid被清除了
            throw new RuntimeException();
        }

        return new SimpleAuthenticationInfo(currentSigner, null, getName());
    }
}
~~~

