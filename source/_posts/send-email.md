---
title: 使用Simple Java Mail 5发送邮件

date: 2018-05-26 07:02

updated: 2018-05-26 07:02

tags:
- 工具类

categories: Java

permalink: 
---



## 简介

在Java中，可以使用Simple Java Mail这个类库来发送E-mail。



## 事前准备

首先要准备一个开启了`POP3/SMTP`服务的邮箱，比如163邮箱。

![](/images/send-email-1.png)



163邮箱启用`POP3/SMTP`时需要设置一个授权密码，记住它，后面会用得到。



## 引入依赖

~~~xml
<dependency>
    <groupId>org.simplejavamail</groupId>
    <artifactId>simple-java-mail</artifactId>
    <version>5.0.3</version>
</dependency>
~~~



## 示例

~~~java
    /**
     * 发送邮件
     *
     * @param emails 收件人E-mail，支持群发
     * @param subject 邮件主题
     * @param content 邮件正文
     */
    public void sendEmail(List<String> emails, String subject, String content) {
        EmailPopulatingBuilder builder = EmailBuilder.startingBlank().from("Demo App Robot", "demo_app_robot@163.com").withSubject(
                subject).withPlainText(content);
        for (String email : emails) {
            builder.to(email.split("@")[0], email);
        }
        Email email = builder.buildEmail();
        MailerBuilder.withSMTPServer("smtp.163.com", 25, "demo_app_robot@163.com", "demoapp233").buildMailer().sendMail(email);
    }
~~~



`"smtp.163.com"`和`25`是由163邮箱决定的

`"demo_app_robot@163.com"`代表事前准备的邮箱地址

`"demoapp233"`代表邮箱启用`POP3/SMTP`时设置的授权密码**（而不是注册邮箱时的登录密码）**

这些参数可以以配置的方式引入。



至此，可以使用调用`sendEmail()`方法发送邮件了，Simple Java Mail所能做的远远不止示例中演示的那些，想要了解更多可以参考[官方文档](http://www.simplejavamail.org/#/features)。