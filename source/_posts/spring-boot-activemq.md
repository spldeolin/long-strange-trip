---
title: Spring Boot集成activeMQ

date: 2018-05-13 08:50:00

updated: 2018-05-13 08:50:00

tags:
- Spring Boot

categories: Java

permalink: spring-boot-activemq
---

## 简介

通过消息机制实现最终一致性，是针对分布式事务的一种比较简单的实践方式。而在Spring Boot项目中，activeMQ是集成起来最方便的消息队列了。



## Windows下安装activeMQ

1. 官网

   http://activemq.apache.org/download-archives.html

2. 解压后运行`%activemq%\bin\activemq.bat`



## POM追加依赖

~~~xml
<!--springboot activemq-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-activemq</artifactId>
</dependency>
~~~



## Application.yml

~~~yaml
spring:

  activemq:
    broker-url: tcp://127.0.0.1:61616
    spring.activemq.user: admin
    spring.activemq.password: admin
~~~

其中，`spring.activemq.broker-url`中的端口号必须与`%activemq%\conf\activemq.xml`中，`<transportConnector name="openwire"/>`保持一致。



## 配置队列

~~~java
@Configuration
public class ActivemqConfig {

    @Bean("oneCookieQueue")
    public Queue queue() {
        return new ActiveMQQueue("onecookie.queue");
    }

}
~~~



## 生产者消费者

~~~java
@Component
@Log4j2
public class Producer {

    @Autowired
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Autowired
    @Qualifier("oneCookieQueue")
    private Queue queue;

    @Scheduled(fixedDelay = 3000)
    public void send() {
        log.info("准备发送一个曲奇饼干");
        this.jmsMessagingTemplate.convertAndSend(this.queue, "队列中的曲奇饼干");
    }

}
~~~



~~~java
@Component
@Log4j2
public class Customer {

    @JmsListener(destination = "onecookie.queue")
    public void receiveQueue(String text) {
        log.info("收到曲奇饼干：{}", text);
    }

}
~~~



## 发送对象

如果想发送对象的话，需要把`Trusted Package`启用。

~~~java
    @Autowired
    private ActiveMQConnectionFactory activeMQConnectionFactory;

    @PostConstruct
    public void setTrustedPackages() {
        activeMQConnectionFactory.setTrustAllPackages(true);
    }
~~~



或是

~~~java
    @Autowired
    private ActiveMQConnectionFactory activeMQConnectionFactory;

    @PostConstruct
    public void setTrustedPackages() {
        activeMQConnectionFactory.setTrustedPackages(Lists.newArrayList("com.spldeolin.beginningmind"));
    }
~~~

