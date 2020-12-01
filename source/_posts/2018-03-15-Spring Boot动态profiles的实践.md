---
title: Spring Boot动态profiles的实践

date: 2018-03-15 16:55

updated: 2018-03-15 16:55

tags:
- Spring Boot

categories: Java

permalink: springboot-active-profiles
---

### Spring Boot的Profiles属性

Spring Boot中有一个profiles的概念，解决了不同环境需要不同配置的问题。



项目中一般会有多个application.yml文件，`application.yml` 可以认为是主配置文件，其他的`application-dev.yml` `application-test.yml`......可以认为是子配置文件。

主配置文件中，会定义`spring: profiles: active: `属性，用于指定某个子配置文件。

子配置文件中，会定义`spring: profiles: `属性，用于指定自己是哪个子配置文件。

被指定的子配置文件中的其他配置，将会覆盖掉主配置文件中的同名属性。可以参考下面的例子



```yaml
# application.yml配置文件
spring:
  profiles:
    active: dev
	
custom-properties:
  one-cookie: 被覆盖的曲奇饼干
```

```yaml
# application-dev.yml配置文件
spring:
  profiles: dev

custom-properties:
  one-cookie: 开发中的曲奇饼干
```

```yaml
# application-prod.yml配置文件
spring:
  profiles: prod

custom-properties:
  one-cookie: 生产中的曲奇饼干
```

```java
// 请求方法：测试项目启动后，配置文件中custom-properties: one-cookie:属性值是什么

@Value("${custom-properties.one-cookie}")
private String oneCookie;

@GetMapping("test")
public String test() {
    return this.oneCookie;
}
```

访问`http://localhost/test`将会得到`开发中的曲奇饼干`。



### Maven的profiles标签

到目前为止，可以看到，Spring Boot的`spring: profiles: active:`必须是硬编码的，每次不同环境打包/调试时，都要主动去改这个属性，显得非常的不方便。



现在，需要引入Maven的profiles标签，来实现动态指定profiles的效果。



如下所示，在pom.xml中追加profiles标签。

~~~xml
<!--pom.xml-->
<profiles>
   		<!--dev环境-->
        <profile>
            <id>dev</id>
            <properties>
                <profiles.active>dev</profiles.active>
            </properties>
            <!--默认为dev-->
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
        </profile>
    	<!--prod环境-->
        <profile>
            <id>prod</id>
            <properties>
                <profiles.active>prod</profiles.active>
            </properties>
        </profile>
    </profiles>
</profiles>
~~~



并将`application.yml`的`spring: profiles: active: `从硬编码更改为与properties标签中的自定义标签一致的占位符。

~~~yaml
# application.yml配置文件
spring:
  profiles:
    active: @profiles.active@
	
custom-properties:
  one-cookie: 被覆盖的曲奇饼干
~~~

至此，配置完成。



### 打包方式

通过以下的命令行进行打包

```cmd
mvn clean install -Pprod
```



`-Pprod`将会找到id为`prod`所在profile标签中的profiles.active标签，并用标签值替换掉`application.yml`的`@profiles.active@`占位符。

虽然所有的application.yml文件都会被打进包内，但是`application.yml`只会指定`prod`作为profile。

这里的`-Pprod`可以不指定，那样做的话将会缺省为`activeByDefault`为`true`的`profile`



### 调试方式A

可以通过以下的命令行进行打包

```cmd
mvn spring-boot:run -Pprod
```

与`clean install`同理。

这个方式需要项目有Maven插件`spring-boot-maven-plugin`



### 调试方式B

对于那种项目中有多个启动方法的微服务项目，`spring-boot:run`命令可能会有问题。

这类项目一般用main方法调试更方便，那么也可以通过在main方法后面追加一下的Java参数来指定profiles

~~~
-Dspring.profiles.active=dev
~~~



这种方式**无法缺省**，一旦不指定`-Dspring.profiles.active`，就会报错，因为它不是Maven命令行，无法使用定义在Maven的pom.xml中缺省值。