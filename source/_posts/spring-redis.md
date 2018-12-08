---
title: Spring Data Redis的集成与Redis API

date: 2018-01-30 08:46:00

updated: 2018-01-30 08:46:00

tags:
- Spring
- Redis

categories: Java

permalink: spring-redis
---

## 简介

这篇POST介绍的是如何在Spring中集成Redis，同时也会介绍了一个用于便捷操作Redis的自定义API类。

在实现上，选择`Protostuff`作为**序列化**框架，选择`Objenesis`作为序列化过程中的**实例化**框架。

## pom追加依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.8.10.RELEASE</version>
</dependency>
<!--protostuff-->
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-core</artifactId>
    <version>1.1.3</version>
</dependency>
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-runtime</artifactId>
    <version>1.1.3</version>
</dependency>
<!--objenesis-->
<dependency>
    <groupId>org.objenesis</groupId>
    <artifactId>objenesis</artifactId>
    <version>2.6</version>
</dependency>
```

## 创建spring-redis.xml

（Redis配置文件的创建与引入省略）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="jedisConnFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="poolConfig" ref="jedisPoolConfig" />
        <property name="hostName" value="${redis.host}" />
        <property name="port" value="${redis.port}" />
        <property name="timeout" value="${redis.timeout}" />
        <property name="password" value="${redis.password}" />
        <property name="database" value="${redis.dbindex}" />
    </bean>

    <bean id="redisTemplate"
            class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="jedisConnFactory" />
        <property name="keySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
        </property>
    </bean>

    <bean class="com.spldeolin.djr.cache.RedisCache" />

</beans>
```

## SerializationUtil

~~~java
/**
 * 序列化工具
 */
public class SerializationUtil {

    private static Map<Class<?>, Schema<?>> cachedSchema = new ConcurrentHashMap<>();

    private static Objenesis objenesis = new ObjenesisStd(true);

    private static <T> Schema<T> getSchema(Class<T> cls) {
        @SuppressWarnings("unchecked")
        Schema<T> schema = (Schema<T>) cachedSchema.get(cls);
        if (schema == null) {
            schema = RuntimeSchema.createFrom(cls);
            cachedSchema.put(cls, schema);
        }
        return schema;
    }

    public static <T> byte[] serialize(T obj) {
        @SuppressWarnings("unchecked")
        Class<T> cls = (Class<T>) obj.getClass();
        LinkedBuffer buffer = LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE);
        try {
            Schema<T> schema = getSchema(cls);
            return ProtostuffIOUtil.toByteArray(obj, schema, buffer);
        } catch (Exception e) {
            throw new IllegalStateException(e.getMessage(), e);
        } finally {
            buffer.clear();
        }
    }

    public static <T> T deserialize(byte[] data, Class<T> cls) {
        try {
            T message = objenesis.newInstance(cls);
            Schema<T> schema = getSchema(cls);
            ProtostuffIOUtil.mergeFrom(data, message, schema);
            return message;
        } catch (Exception e) {
            throw new IllegalStateException(e.getMessage(), e);
        }
    }

}
~~~

选择`objenesis.newInstance(cls)`而不是JDK自带的`cls.newInstance()`来实例化对象的原因是，

后者遇到那些没有无参构造方法的类（如Integer等）会直接报错。

## RedisCache

```java
/**
 * 存取Redis缓存的API
 */
@Component
public class RedisCache {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    /**
     * 存入一个缓存
     */
    public void put(String key, Object obj) {
        final byte[] rawKey = key.getBytes();
        final byte[] rawValue;
        if (obj instanceof String) {
            rawValue = ((String) obj).getBytes();
        } else {
            rawValue = SerializationUtil.serialize(obj);
        }
        redisTemplate.execute(new RedisCallback<Object>() {
            @Override
            public Object doInRedis(RedisConnection connection) throws DataAccessException {
                connection.set(rawKey, rawValue);
                return null;
            }
        });
    }

    /**
     * 存入一个<b>不存在的缓存</b>，如果key已存在，则什么都不发生
     */
    public boolean putNX(String key, Object obj) {
        final byte[] rawKey = key.getBytes();
        final byte[] rawValue;
        if (obj instanceof String) {
            rawValue = ((String) obj).getBytes();
        } else {
            rawValue = SerializationUtil.serialize(obj);
        }
        boolean result = redisTemplate.execute(new RedisCallback<Boolean>() {
            @Override
            public Boolean doInRedis(RedisConnection connection) throws DataAccessException {
                return connection.setNX(rawKey, rawValue);
            }
        });
        return result;
    }

    /**
     * 存入一个将会失效的缓存，单位是<b>秒</b>
     */
    public <T> void putWithExpire(String key, T obj, final long expireTime) {
        final byte[] rawKey = key.getBytes();
        final byte[] rawValue;
        if (obj instanceof String) {
            rawValue = ((String) obj).getBytes();
        } else {
            rawValue = SerializationUtil.serialize(obj);
        }
        redisTemplate.execute(new RedisCallback<Boolean>() {
            @Override
            public Boolean doInRedis(RedisConnection connection) throws DataAccessException {
                connection.setEx(rawKey, expireTime, rawValue);
                return true;
            }
        });
    }

    /**
     * 取出一个缓存
     */
    public <T> T get(final String key, Class<T> clazz) {
        byte[] result = redisTemplate.execute(new RedisCallback<byte[]>() {
            @Override
            public byte[] doInRedis(RedisConnection connection) throws DataAccessException {
                return connection.get(key.getBytes());
            }
        });
        if (result == null) {
            return null;
        }
        if (clazz == String.class) {
            return (T) new String(result);
        } else {
            return SerializationUtil.deserialize(result, clazz);
        }
    }

    /**
     * 删除一个缓存
     */
    public void delete(String key) {
        redisTemplate.delete(key);
    }

    /**
     * 得到一个缓存的剩余失效时间
     */
    public Long getExpire(String key) {
        return redisTemplate.getExpire(key);
    }

}
```

特别要注意的94行和101行的两个方法调用。在这两个地方，`RedisCache`并没有将`key`转化为`byte[]`类型，而是直接移交给`redisTemplate`对象处理，而在`redisTemplate`对象内部，会对`key`进行`rawKey()`处理，`rawKey()`方法如下：

```java
	@SuppressWarnings("unchecked")
	private byte[] rawKey(Object key) {
		Assert.notNull(key, "non null key required");
		if (keySerializer == null && key instanceof byte[]) {
			return (byte[]) key;
		}
		return keySerializer.serialize(key);
	}
```

他会使用在`spring-redis.xml`中配置的`StringRedisSerializer`策略，来序列化`key`

> Key往往是String，这样做的话大部分Redis可视化工具都能清晰地展示出Key原来的内容，可读性比较强。

所以在`RedisCache`的其他方法中，Deolin也采用了与`StringRedisSerializer`同样的方式来序列化`key`，即`key.getBytes()`。