---
title: Spring Boot 配置、Profiles、调试、打包问题总结

date: 2018-04-11 11:55

updated: 2018-04-11 11:55

tags:
- Spring Boot
- 总结

categories: Java

permalink: spring-boot-summary
---

## 简介

这篇POST汇总了Spring Boot开发过程中*配置*、*Profiles*、*调试*、*打包*几个方面的各种问题。

Spring Boot版本为`1.5.12`，Maven版本为`3.5.2`，IDE为`IntelliJ IDEA`

## application配置

Spring Boot的配置文件是`application.yml`，项目的所有配置都在这里面。如果集成了Log4j2的话，还需要`log4j2.yml`。

> 为了让配置看上去比较整洁，Deolin选择yaml作为配置文件的格式。

如果仅仅是`application.yml`和`log4j2.yml`的话，仅仅需要放在`src/main/resources`目录下，不需要任何额外的指定，Spring Boot能自己扫描到它们，并引入其中的属性。

## 业务中引用application属性值

想要引用属性值，需要用一个Bean来映射这些属性，映射方式有两种

1. ConfigurationProperties方式

~~~java
@Component
@ConfigurationProperties(prefix = "global-properties")
@Data
public class GlobalProperties {
    
 	private String oneCookie;
    
	private Boolean oneFlag;
    
}
~~~

2. Value方式

~~~java
@Component
@Data
public class GlobalProperties {
    
    @Value("${global-properties.one-cookie}")
    private String oneCookie;
    
    @Value("${global-properties.one-flag}")
    private String oneFlag;
    
}

~~~

建议第一种方式，代码量更少，而且第二种方式的属性只支持String类型



## 使用log4j2.yml

Log4j2的Starter默认是不认识yaml格式的文件的，所以需要追加一个用于提供支持的依赖

~~~xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
    <version>2.9.4</version>
</dependency>
~~~



## 启动项目

启动方式有两种，main方法启动或是Maven命令行启动。开发过程中推荐main方法启动

### main方法启动

Spring Initializr会直接为项目生成入口类，直接用IDE运行这个类即可。

这种方式不用走Maven，建议使用这种方式。

### Maven命令行启动

Spring Boot默认应该配置Maven插件`spring-boot-maven-plugin`，可以用IDE运行以下Maven命令来启动项目。

~~~shell
$ mvn springboot:run
~~~



## 动态化配置

上面提到了两个配置文件，`application.yml`和`log4j2.yml`，这里的所有配置，都是可以通过Java的命令行参数来动态指定。

### 命令行参数

Java程序在运行时，可以指定一些参数，对应的是main方法的那个String数组参数，指的就是这一命令行参数

对于一个main方法，在IntelliJ IDEA的指定`VM Options`即可，格式如下

![](/images/2018041101.png)

对于一个Maven启动命令，直接在命令后面追加参数就可以了

![](/images/2018041102.png)

### 动态指定application.yml内属性

介绍命令行参数，是因为它可以覆盖掉`application.yml`里的属性，以下面为例子

~~~yaml
global-properties:
	one-cookie: 一块曲奇饼干
~~~

如果我在启动时（无论是main方法还是Maven命令的方式）指定了参数`global-properties.one-cookie=命令行里的曲奇饼干`，那么启动后，Spring容器中的`global-properties.one-cookie`的值，会是`命令行里的曲奇饼干`。

> 可以通过@Value注解在控制器引入`${global-properties.one-cookie}`，来验证上述结论。

这样一来，application配置完全可以实现动态指定的效果。

### 动态指定log4j2.yml内属性

命令行参数除了覆盖application配置，还能覆盖log4j2的配置。以下面为例子

~~~yaml
Configuration:
  status: warn

  Appenders:
    Console:
      name: console
      target: SYSTEM_OUT
      ThresholdFilter:
        level: info         # 修改为“debug”可查看DEBUG级日志
        onMatch: ACCEPT
        onMismatch: DENY
      PatternLayout:
        charset: utf-8
        pattern: "%d{HH:mm:ss} %4p [%t][%file:%L] - %m%n"

  Loggers:
    Root:
      level: all
      AppenderRef:
        - ref: console
~~~

这是一个简单配置文件，效果是在控制台打印INFO及以上级别的log。

现在，作出如下改动

~~~yaml
Configuration:
  status: warn

  Properties:
    Property:
    - name: log.level
      value: info

  Appenders:
    Console:
      name: console
      target: SYSTEM_OUT
      ThresholdFilter:
        level: ${sys:log.level}
        onMatch: ACCEPT
        onMismatch: DENY
      PatternLayout:
        charset: utf-8
        pattern: "%d{HH:mm:ss} %4p [%t][%file:%L] - %m%n"

  Loggers:
    Root:
      level: all
      AppenderRef:
        - ref: console
~~~

可以看到一个占位符`${sys:log.level}`。它代表的意思是

​	*如果命令行参数没有指定了log.level，那么缺省为`Configuration.Properties.Property.name`为log.level所对应的`value`*。



## 多Profiles

动态化配置固然很方便，但随着配置增多，启动命令会越来越长，开发人员也会越来越不好管理这些动态配置。对于动态配置，更优雅的做法是把配置整理起来，做成多个Profiles，以供选择。Deolin在这里提供了一种实践方式（不用于演示的配置将会省略掉）

------

`src/main/resources/application.yml`

~~~yaml
spring:

  profiles.active: ${profile} # 每次写-Dspring.profiles.active太长了，这里这样写可以替换成-Dprofile

global-properties:

  one-cookie: 缺省的曲奇饼干

  one-flag: false
~~~

`src/main/resources/application-dev.yml`

~~~yaml
global-properties:

  one-cookie: 开发中的曲奇饼干

  one-flag: true

logging.config: classpath:log4j2/log4j2-dev.yml
~~~

`src/main/resources/application-prod.yml`

~~~yaml
global-properties:

  one-cookie: 投入生产的曲奇饼干

  one-flag: true

logging.config: classpath:log4j2/log4j2-prod.yml
~~~

`src/main/resources/log4j2/log4j2-dev.yml`**（这里多包了一层文件夹，不这样做的话，不同profiles的log4j2配置会产生一些现象很奇怪的冲突，目前只能通过这个方式来解决）**

~~~yaml
# 适用于开发环境的Log4j2配置，具体省略
~~~

`src/main/resources/log4j2/log4j2-prod.yml`

~~~yaml
# 适用于生产环境的Log4j2配置，具体省略
~~~

------

如此一来，即便有很多的配置属性，也只需要在启动时指定`-Dprofile`，就能指定一批配置。

> Deolin不太建议Maven的profiles标签来管理Spring的profiles，这样做需要多写很多配置。

## 单体测试

如果新写了一个Service类、DAO类或者别的什么Spring组件，强烈推荐用Test starter提供的方式来测试它们。

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("dev")
@Log4j2
public class Tests {

    @Autowired
    private FrostboltService frostboltService;

    @Test
    public void test() {
        frostboltService.launch("基尔加丹");
    }

}
~~~

然后直接用JUnit方式启动测试类即可。

值得一提的是，这里也是支持命令行参数指定profile的，但如果你觉得每新建一个Test类都要写命令行参数太麻烦，可以直接像上面的例子那样声明一个`@ActiveProfiles`注解。（毕竟单体测试一般都是开发环境做的事情）

## 打包

### jar包

将pom的`package`标签的内容改为`jar`，然后运行Maven命令。

~~~shell
$ mvn clean package
~~~

### war包

这里不再演示如何打成war包了，因为需要配置额外的东西，Deolin认为这样做违背了Spring Boot开袋即食的风格。

[这篇文章](https://dzone.com/articles/what-archive-format-should-you-use-war-or-jar)中也建议打成jar包。



## 部署项目

### 部署jar

直接运行jar文件

~~~shell
java -jar -Dprofile=prod summary.jar
~~~

### 部署war

不再演示



## 演示项目

[spldeolin](https://github.com/spldeolin) / [spring-boot-summary-demo](https://github.com/spldeolin/spring-boot-summary-demo)