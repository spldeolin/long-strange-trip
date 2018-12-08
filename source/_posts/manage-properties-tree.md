---
title: 管理application.yml中树状结构的属性

date: 2018-04-12 14:59:00

tags:
- Spring Boot

categories: Java

permalink: manage-properties-tree
---

## 场景

随着Spring Boot项目扩张，`application.yml`里的自定义属性会越来越多，并呈现出树状结构，如下所示

~~~yaml
# 非自定义属性省略

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
  use-one-cent-when-pay: true

  # 中文简体文案
  text-zh-hans:
    hello: 你好
    excellent: 优秀

  # 日语文案
  text-ja-jp:
    hello: こんにちは
    excellent: 素晴らしい

  # 微信（第三方）
  wechat:
    appid: asdfas22
    appsecret: asdfa232

  # 阿里大鱼（第三方）
  alidayu:
    appid: asdasd
    appsecret: asdfasdf
    sign-name: dssd
    template-code: sdfsd
~~~

一般来说，做好归纳工作的话，自定义属性的树状深度也就1~2层左右。

## 方案

创建一个总的属性类，创建多个子分类属性类。

~~~java
/**
 * 配置一览
 */
@Component
@ConfigurationProperties(Properties.PREFIX)
@Data
public class Properties {

    public static final String PREFIX = "beginning-mind";

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
     * 使用一分钱支付
     */
    private Boolean useOneCentWhenPay;

    /**
     * 时间格式配置
     */
    @Autowired
    private TimeProperties timeProperties;

    /**
     * 中文简体文案配置
     */
    @Autowired
    private TextZhHansProperties textZhHansProperties;

    /**
     * 日语文案配置
     */
    @Autowired
    private TextJaJpProperties textJaJpProperties;

    /**
     * 微信配置（第三方）
     */
    @Autowired
    private WechatProperties wechatProperties;

    /**
     * 阿里大鱼配置（第三方）
     */
    @Autowired
    private AlidayuProperties alidayuProperties;

}
~~~

~~~java
@Component
@ConfigurationProperties(Properties.PREFIX + ".text-ja-jp")
@Data
public class TextJaJpProperties {

    private String hello;

    private String excellent;

}
~~~

~~~java
@Component
@ConfigurationProperties(Properties.PREFIX + ".alidayu")
@Data
public class AlidayuProperties {

    private String appid;

    private String appsecret;

    private String signName;

    private String templateCode;

}
~~~

> ...考虑到篇幅，其他子分类的属性类省略

这种方案要写的样板代码比较少，层级关系也完整的映射到了Java类中，最终，Properties类的对象在Spring容器中是这样一个状态

![](/images/2018041201.png)



业务代码可以通过`@Autowired` `Properties`为入口，获取所有的需要的属性。

~~~java
	@Autowired
	private Properties properties;

	public String getSmsTemplate() {
        return properties.getAlidayuProperties().getTemplateCode();
	}
~~~

