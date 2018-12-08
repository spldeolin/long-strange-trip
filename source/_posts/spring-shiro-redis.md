title: Shiro + Redis集成思路
date: 2017-11-24 14:10:00

tags:

- Spring
- Shiro
- Redis

categories: Java

permalink: spring-shiro-redis
---

## 简介

这篇POST介绍了`Spring / SpringMVC`环境下，集成`Shiro`后，再集成`Redis`，以实现分布式缓存与分布式会话功能的思路，主要步骤是先集成Shiro，再集成Redis，最后让Shiro与Redis集成起来。

## Spring集成Shiro

### 追加依赖

~~~xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.3.2</version>
</dependency>
~~~

### 注册Bean

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myShiroRealm" class="io.spldeolin.bestpractice.shiro.component.Realm">
        <property name="cacheManager" ref="cacheManager" />
    </bean>

    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="myShiroRealm" />
        <property name="cacheManager" ref="cacheManager" />
        <property name="sessionManager" ref="sessionManager" />
    </bean>

    <bean id="sessionManager" class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
        <property name="sessionIdUrlRewritingEnabled" value="false" />
        <property name="globalSessionTimeout" value="3600000" />
    </bean>

    <!-- 自带的、缓存在内存的、不支持集群的缓存管理器 -->
    <bean id="cacheManager" class="org.apache.shiro.cache.MemoryConstrainedCacheManager" />

    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityManager" />
        <!-- loginUrl 未认证时，访问需要认证资源时的重定向url -->
        <property name="loginUrl" value="/" />
        <!-- successUrl 登录成功后的重定向url -->
        <property name="successUrl" value="/loginsuccess.jhtml" />
        <!-- unauthorizedUrl 访问无权限资源时的重定向url -->
        <property name="unauthorizedUrl" value="/error.jhtml" />
        <property name="filterChainDefinitions">
            <value>
                /** = anon
            </value>
        </property>
    </bean>

    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />

    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
            depends-on="lifecycleBeanPostProcessor">
        <property name="proxyTargetClass" value="true" />
    </bean>

    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager" />
    </bean>

</beans>
~~~

可以看到，有三个主要的组件，`Realm`、`SecurityManager`、`ShiroFilter`。

`Realm`代表取得“用于验证和授权的数据”的策略。

`SecurityManager`持有`Realm`、`CacheManager`、`SessionManager`对象。后两者代表缓存策略和会话管理策略，演示代码暂时采取Shiro默认策略。

`ShiroFilter`代表过滤器，为web.xml中配置的过滤器提供支持。

### web.xml配置过滤器

~~~xml
<filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>*</url-pattern>
    </filter-mapping>
~~~

### 实现涉及到的自定义类

上述示例代码中，只需要实现Realm。这里省略



## Spring集成Redis

### 追加依赖

~~~xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.7.2</version>
</dependency>
~~~

### 注册Bean

~~~xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="redisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxTotal" value="${redis.pool.maxActive}" />
        <property name="maxIdle" value="${redis.pool.maxIdle}" />
        <property name="maxWaitMillis" value="${redis.pool.maxWait}" />
        <property name="testOnBorrow" value="${redis.pool.testOnBorrow}" />
    </bean>

    <bean id="jedisPool" class="redis.clients.jedis.JedisPool"
        destroy-method="destroy">
        <constructor-arg ref="redisPoolConfig" />
        <constructor-arg value="${redis.host}" />
        <constructor-arg type="int" value="${redis.port}" />
        <constructor-arg type="int" value="${redis.timeout}" />
        <constructor-arg type="java.lang.String" value="${redis.password}" />
        <constructor-arg type="int" value="${redis.dbindex}" />
    </bean>

    <bean id="redisClient" class="io.spldeolin.logindemo.shiro.component.RedisClient4Shiro">
        <constructor-arg name="jedisPool" ref="jedisPool" />
        <property name="expire" value="${redis.default.expire}" />
    </bean>

</beans>
~~~

## Shiro与Redis集成

### 追加依赖

~~~xml
<dependency>
    <groupId>org.crazycake</groupId><!--这是一个第三方依赖，引入它以后可以少写很多用于Shiro Redis之间互相集成的组件-->
    <artifactId>shiro-redis</artifactId>
    <version>2.4.2.1-RELEASE</version>
</dependency>
~~~

### 修改Shiro的`SessionManager`

~~~xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="realm" />
        <property name="cacheManager" ref="cacheManager" />
        <!-- 【securityManager的sessionManager相关】 -->
        <property name="sessionManager">
            <!-- Shiro自带的session管理方式——DefaultWebSessionManager和MemorySessionDAO -->
            <!-- <bean class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager"> 
                <property name="sessionIdUrlRewritingEnabled" value="false" /> <property 
                name="globalSessionTimeout" value="3600000" /> </bean> -->
            <!-- Redis的session管理方式——DefaultWebSessionManager和RedisSessionDAO -->
            <bean
                class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
                <property name="globalSessionTimeout" value="1800000" />
                <property name="sessionValidationInterval"
                    value="1800000" />
                <property name="sessionDAO">
                    <bean class="org.crazycake.shiro.RedisSessionDAO">
                        <property name="redisManager" ref="redisClient" />
                    </bean>
                </property>
                <property name="sessionIdCookie">
                    <bean class="org.apache.shiro.web.servlet.SimpleCookie">
                        <constructor-arg name="name"
                            value="custom.session" />
                        <property name="path" value="/" />
                    </bean>
                </property>
                <property name="sessionIdUrlRewritingEnabled"
                    value="false" />
            </bean>
        </property>
        <!-- 记住我的失效时间，单位是秒 -->
        <property name="rememberMeManager.cookie.maxAge" value="100000"></property>
    </bean>
~~~

这里的关键是`org.crazycake.shiro.RedisSessionDAO`。

### 修改Shiro的`CacheManager`

~~~xml
 <!-- Shiro自带的缓存管理 -->
 <!-- <bean id="cacheManager" class="org.apache.shiro.cache.MemoryConstrainedCacheManager" 
 /> -->
 <!-- Redis的缓存管理 -->
<bean id="cacheManager" class="org.crazycake.shiro.RedisCacheManager">
     <property name="keyPrefix" value="shiro_redis_session:" />
     <property name="redisManager" ref="redisClient" />
</bean>
~~~

这里的关键是`org.crazycake.shiro.RedisCacheManager`。`

同时，`spring-redis.xml`与修改后`spring-shiro.xml`的联系点在`redisClient`这个bean。

### 实现`RedisClient`

~~~java
public class RedisClient4Shiro extends RedisManager {

    private JedisPool jedisPool = null;

    public RedisClient4Shiro(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    @Override
    public void init() {
        super.init();
    }

    @Override
    public byte[] get(byte[] key) {
        Jedis jedis = jedisPool.getResource();
        byte[] value;
        try {
            value = jedis.get(key);
        } catch (Exception e) {
            throw new RuntimeException("redis operation error:", e);
        } finally {
            jedis.close();
        }
        return value;
    }

    @Override
    public byte[] set(byte[] key, byte[] value) {
        Jedis jedis = jedisPool.getResource();
        try {
            jedis.set(key, value);
            Integer expire = getExpire();
            if (expire != 0) {
                jedis.expire(key, expire);
            }
        } catch (Exception e) {
            throw new RuntimeException("redis operation error:", e);
        } finally {
            jedis.close();
        }
        return value;
    }

    @Override
    public byte[] set(byte[] key, byte[] value, int expire) {
        Jedis jedis = jedisPool.getResource();
        try {
            jedis.set(key, value);
            if (expire != 0) {
                jedis.expire(key, expire);
            }
        } catch (Exception e) {
            throw new RuntimeException("redis operation error:", e);
        } finally {
            jedis.close();
        }

        return value;
    }

    @Override
    public void del(byte[] key) {
        Jedis jedis = jedisPool.getResource();
        try {
            jedis.del(key);
        } catch (Exception e) {
            throw new RuntimeException("redis operation error:", e);
        } finally {
            jedis.close();
        }
    }

    @Override
    public void flushDB() {
        Jedis jedis = jedisPool.getResource();
        try {
            jedis.flushDB();
        } catch (Exception e) {
            throw new RuntimeException("redis operation error:", e);
        } finally {
            jedis.close();
        }
    }

    @Override
    public Long dbSize() {
        Long dbSize = Long.valueOf(0L);
        Jedis jedis = jedisPool.getResource();

        try {
            dbSize = jedis.dbSize();
        } catch (Exception e) {
            throw new RuntimeException("redis operation error:", e);
        } finally {
            jedis.close();
        }
        return dbSize;
    }

    @Override
    public Set<byte[]> keys(String pattern) {
        Set<byte[]> keys = null;
        Jedis jedis = jedisPool.getResource();

        try {
            keys = jedis.keys(pattern.getBytes());
        } catch (Exception e) {
            throw new RuntimeException("redis operation error:", e);
        } finally {
            jedis.close();
        }
        return keys;
    }

}
~~~

## 示例源码

[spldeolin](https://github.com/spldeolin) / [**login-demo**](https://github.com/spldeolin/login-demo)

