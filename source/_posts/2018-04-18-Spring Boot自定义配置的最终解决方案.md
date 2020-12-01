---
title: Spring Boot自定义配置的最终解决方案

date: 2018-04-18 13:09

updated: 2018-04-18 13:09

tags:
- Spring Boot

categories: Java

permalink: springboot-properties
---

## 简介

这篇POST介绍了以下问题的最终解决方案

- 如何在`application.yml`中，自定义各种结构的配置（从简单到复杂）
- 如何将`application.yml`的配置映射到Java类
- 如何在业务代码中获取这些配置的值

这篇POST将会以beginning-mind项目为示例，为了能说明问题，不演示多profiles的问题。



## 简单的字符配置

`application.yml`

~~~yaml
# 其他Spring Boot基本配置和各个Starter的配置，全部省略

beginning-mind:

  one-cookie: 一个曲奇饼干

~~~

`Properties.java`

~~~java
/**
 * 配置一览
 */
@Component
@ConfigurationProperties(value = "beginning-mind")
@Data
public class Properties {
    
    /**
     * “一个曲奇”
     */
    private String oneCookie;
    
}
~~~



## 简单的数字、布尔型配置

`application.yml`

~~~yaml
beginning-mind:

  one-cookie: 一个曲奇饼干
  
  version: 233
  
  price: 9.15
  
  use-one-cent-when-pay: true

~~~

`Properties.java`

~~~java
@Component
@ConfigurationProperties(value = "beginning-mind")
@Data
public class Properties {
    
    private String oneCookie;
    
    private Integer version;
    
    private BigDecimal price;
    
    private Boolean useOneCentWhenPay;
    
}
~~~



## 子配置

`application.yml`

~~~yaml
beginning-mind:
  one-cookie: 一个曲奇饼干
  version: 233
  price: 9.15
  use-one-cent-when-pay: true
  
  text-zh-hans:

  	hello: 你好

  	excellent: 优秀

~~~

`Properties.java`

~~~java
@Component @ConfigurationProperties(value = "beginning-mind") @Data
public class Properties {
    private String oneCookie;
    private Integer version;
    private BigDecimal price;
    private Boolean useOneCentWhenPay;
    
    private TextZhHans textZhHans;
    
    @Data
    public static class TextZhHans {

        private String hello;

        private String excellent;

    }

}
~~~



## 简单的集合型构造配置

`application.yml`

~~~yaml
beginning-mind:
  one-cookie: 一个曲奇饼干
  version: 233
  price: 9.15
  use-one-cent-when-pay: true
  text-zh-hans:
  	hello: 你好
  	excellent: 优秀

  struct:
    array:
      strings:
      - a
      - b
      - c
      integers:
      - 0
      - 10
      - 999
~~~

`Properties.java`

~~~java
@Component @ConfigurationProperties(value = "beginning-mind") @Data
public class Properties {
    private String oneCookie;
    private Integer version;
    private BigDecimal price;
    private Boolean useOneCentWhenPay;
    private TextZhHans textZhHans;
    @Data public static class TextZhHans {
        private String hello;
        private String excellent;
    }

    @Data
    public static class Struct {
        
        private Array array;
    
        @Data
        public static class Array {

            private List<String> strings;

            private List<Integer> integers;

        }

}
~~~



## Map型与集合型复合构造配置

这应该是最复杂的情况了，可能的话，借助**直尺**来观看下面的`application.yml`

~~~yaml
beginning-mind:

  one-cookie: 一个曲奇饼干

  time:
    default-date-pattern: yyyy-MM-dd
    default-time-pattern: HH:mm:ss
    default-datetime-pattern: yyyy-MM-dd HH:mm:ss
    serialize-java-util-date-to-timestamp: true

  # 未分类
  ip: sssad
  version: 233
  price: 9.15
  use-one-cent-when-pay: true

  # 中文简体文案
  text-zh-hans:
    hello: 你好
    excellent: 优秀

  # 日语文案
  text-ja-jp:
    hello: こんにちは
    excellent: 素晴らしい

  # 微信
  wechat:
    appid: asdfas22
    appsecret: asdfa232

  # 阿里大鱼
  alidayu:
    appid: asdasd
    appsecret: asdfasdf
    sign-name: dssd
    template-code: sdfsd

  struct:
    array:
      strings:
      - a
      - b
      - c
      integers:
      - 0
      - 10
      - 999

    key-value-array:
      high-caloric-food:
      - name: cola
        calorie: 100000000
      - name: chicken
        calorie: 99999
      low-caloric-food:
      - name: cucumber
        calorie: 0.1
      - name: tomato
        calorie: 20
~~~

`Properties.java`

~~~java
/**
 * 配置一览
 */
@Component
@ConfigurationProperties(value = "beginning-mind")
@Data
public class Properties {

//    private static Properties properties;

    /**
     * “一个曲奇”
     */
    private String oneCookie;

    /**
     * IP地址
     */
    private String ip;

    /**
     * 版本号
     */
    private Integer version;

    /**
     * 价格
     */
    private BigDecimal price;

    /**
     * 使用一分钱支付
     */
    private Boolean useOneCentWhenPay;

    /**
     * “时间”格式
     */
    private Time time;

    /**
     * 中文文案
     */
    private TextZhHans textZhHans;

    /**
     * 日语文案
     */
    private TextJaJp textJaJp;

    /**
     * 微信
     */
    private Wechat wechat;

    /**
     * 阿里大鱼
     */
    private Alidayu alidayu;

    /**
     * 各种类型的构造体
     */
    private Struct struct;

    @Data
    public static class Time {

        private String defaultDatePattern;

        private String defaultTimePattern;

        private String defaultDatetimePattern;

        private Boolean serializeJavaUtilDateToTimestamp;

    }

    /**
     * 中文简体文案
     */
    @Data
    public static class TextZhHans {

        private String hello;

        private String excellent;

    }

    /**
     * 日语文案
     */
    @Data
    public static class TextJaJp {

        private String hello;

        private String excellent;

    }

    /**
     * 微信
     */
    @Data
    public static class Wechat {

        private String appid;

        private String appsecret;

    }

    /**
     * 阿里大鱼
     */
    @Data
    public static class Alidayu {

        private String appid;

        private String appsecret;

        private String signName;

        private String templateCode;

    }

    /**
     * 数组/Map性质的构造
     */
    @Data
    public static class Struct {

        /**
         * 简单集合型构造
         */
        private Array array;

        /**
         * Map集合型勾结
         */
        private KeyValueArray keyValueArray;

        @Data
        public static class Array {

            private List<String> strings;

            private List<Integer> integers;

        }

        /**
         * Map列表型构造
         */
        @Data
        public static class KeyValueArray {

            private List<CaloricFood> highCaloricFood;

            private List<CaloricFood> lowCaloricFood;

            @Data
            public static class CaloricFood {

                private String name;

                private BigDecimal calorie;

            }

        }

    }

}
~~~



## 在Bean中获取配置的值

由于`Properties`类是一个`@Component`，所以，任何被`@Controller`、`@Service`、`@Component`、`@Repository`声明的类，内部都可以直接注入`Properties`对象。

~~~java
@Controller
public class UserServiceImpl implements UserService {
    
    @Autowired
    private Properties properties;
    
    @GetMapping
    public String method() {
        return properties.getOneCookie();
    }
    
}
~~~



## 在工具类的静态方法中获取配置的值

工具类往往是在Spring容器外，所以需要借助[ApplicationContext](https://github.com/spldeolin/beginning-mind/blob/master/src/main/java/com/spldeolin/beginningmind/component/ApplicationContext.java)来静态获取容器的`Properties`对象。

~~~java
/**
 * 打招呼工具类
 */
@UtilityClass
@Log4j2
public GreetingUtil {
    
    public void greet() {
        log.info(ApplicationContext.getBean(Properties.class).getTextJaJp().getHello());
    }
    
}
~~~

