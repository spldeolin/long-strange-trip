---
title: RedisTemplate集成Protostuff序列化方式

date: 2018-07-25 08:26

updated: 2018-07-25 08:26

tags:
- Redis
- Protostuff

categories: Java

permalink: redis-template-protostuff
---

过时原因：一般用Redisson了

## 简介

Spring Data Redis的`RedisTemplate`类支持将对象存入Redis，存入之前需要先序列化。

Java中有很多种序列化方式，Protostuff是其中性能较为不错的方式。

这篇POST将会介绍如何将RedisTemplate自带的序列化方式重写成Protostuff。



## 编写序列化器

`RedisTemplate`支持`RedisSerializer<T>`的派生类作为序列化器，想要集成Protostuff，需要先编写一个派生类

~~~java
public static class ProtostuffSerializer implements RedisSerializer<Object> {

        private boolean isEmpty(byte[] data) {
            return (data == null || data.length == 0);
        }

        private final Schema<ProtoWrapper> schema;

        private final ProtoWrapper wrapper;

        private final LinkedBuffer buffer;

        public ProtostuffSerializer() {
            this.wrapper = new ProtoWrapper();
            this.schema = RuntimeSchema.getSchema(ProtoWrapper.class);
            this.buffer = LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE);
        }

        @Override
        public byte[] serialize(Object t) throws SerializationException {
            if (t == null) {
                return new byte[0];
            }
            wrapper.data = t;
            try {
                return ProtostuffIOUtil.toByteArray(wrapper, schema, buffer);
            } finally {
                buffer.clear();
            }
        }

        @Override
        public Object deserialize(byte[] bytes) throws SerializationException {
            if (isEmpty(bytes)) {
                return null;
            }

            ProtoWrapper newMessage = schema.newMessage();
            ProtostuffIOUtil.mergeFrom(bytes, newMessage, schema);
            return newMessage.data;
        }

        private static class ProtoWrapper {

            public Object data;
            
        }
    }
~~~



## 注册RedisTemplate到容器中

~~~java
    @Bean
    public RedisTemplate<String, Object> overrideSerializer(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(factory);
        redisTemplate.setEnableTransactionSupport(true);

        StringRedisSerializer keySerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(keySerializer);
        redisTemplate.setHashKeySerializer(keySerializer);

        ProtostuffSerializer valueSerializer = new ProtostuffSerializer();
        redisTemplate.setValueSerializer(valueSerializer);
        redisTemplate.setHashKeySerializer(valueSerializer);
        return redisTemplate;
    }
~~~



## 使用

至此，可以通过注入`RedisTemplate<String, Object>`来存取对象了。

~~~java
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @GetMapping("/set")
    void ln45() {
        redisTemplate.opsForValue().set("oneCookie", CookieCake.builder().name("曲奇蛋糕").quantity(1).build());
    }

    @GetMapping("/get")
    CookieCake ln51() {
        return (CookieCake) redisTemplate.opsForValue().get("oneCookie");
    }
~~~



使用Object作为value的泛型，会需要调用方做显式的类型转换（就像调用原生HttpServletRequest的getAttribute一样）。

想要规避这种写法，可以编写一个`RedisCache`，代理`RedisTempate`对象，并通过传入`Class<T>`对象，将类型转换做在`RedisCache`的内部。

或者，觉得每次都要`opsForValue()`太麻烦，也可以用`RedisCache`封装掉调用细节。