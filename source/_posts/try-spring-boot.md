---
title: 初次使用Spring Boot

date: 2018-03-11 06:00

updated: 2018-03-11 06:00

tags:
- Spring Boot

categories: Java

permalink: try-spring-boot
---



## 源流

Deolin在这一天开始使用Spring Boot了。


## 创建一个Spring Boot项目

Deolin推荐使用官方的`Spring Initializr`来生成Spring Boot项目，这个做法能节省不少时间。	

地址：https://start.spring.io/ ，版本选定为1.5.10，追加Lombok依赖。

生成完毕导入项目之后，打开pom.xml，可以看到两个关键的依赖`spring-boot-starter` `spring-boot-starter-test`和一个关键的插件`spring-boot-maven-plugin`。

## Spring Boot Starter

值得一提的是，artifactId为`spring-boot-starter-***`的依赖，都是Spring官方发布的、开袋即食的Spring Boot拓展依赖，这些依赖的版本号是跟着spring-boot-starter-parent走的，也就是说，不需要在pom中为这些依赖声明version。

artifactId为`***-spring-boot-starter`的依赖，是非Spring官方发布的，往往是其他框架为了集成到Spring Boot而开发的，如mybatis等。

这些Starter的方便之处在于，往往只需要追加一个Starter，之后加上一点点配置，对应的一整块功能就集成完毕了，不用像Spring + SpringMVC在XML中注册必须的bean。

## 转化为Web项目

在pom中追加starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```



至此，配置完成。

这是启动项目也能成功，但是会秒关闭，原因是整个项目没有一个控制器。可以写个控制器测试一下。

~~~java
@RestController
@RequestMapping("demo")
public class DemoController {

    @GetMapping
    public String get() {
        return "hey, there.";
    } 

}

~~~



结果![](/images/1aaa.png)



## 为项目添加日志

Deolin打算沿用Log4j2，在pom中追加spring-boot-starter-log4j2。

~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
~~~

注意，一个项目中只能有一个日志实现，而`spring-boot-starter`内部依赖了logback，想要用log4j2的话，需要使用exclusions标签解除内部依赖。

~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
~~~

最后，为Log4j2写一套配置（*src/main/resources/log4j2.yml*）

建议使用[yaml格式的Log4j2配置](http://spldeolin.com/2018/04/05/log4j2-yml/)。



至此，配置完成。

通过刚刚写的控制器测试一下

结果![](/images/S7ZY.png)

可以看到，不光得到想要的结果，而且Spring Boot的启动日志的格式也遵循了log4j2.xml中定义的日志Layout，这是因为`spring-boot-starter-log4j2`中包含了适配器依赖。



## 测试/调试项目

前面提到Spring Initializr生成的项目默认整合了`spring-boot-starter-test`，也就是说，Deolin可以直接进行单体测试了。

### 调试Bean

对于这类代码，Spring + SpringMVC那时候的做法可能是专门写一个Controller，把目标Bean Autowired进来，然后启动项目，借助http请求来调用Controller，从而调用到目标Bean（这怕是最糟糕的调试方法了，但是Deolin以前确实是这样做的....）

在Spring Boot，情况变得乐观了很多，Spring Initializr为项目生成了测试启动类（src/test/java中），类名是`项目名ApplicationTests`，将它改造一下

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Log4j2
public class BeginningMindApplicationTests {

    private void FrostboltLauncher frostboltLauncher;

    @Test
    public void test1() {
		frostboltLauncher.launch("Deolin");
    }

}
~~~

其中`FrostboltLauncher`是个简单的Bean

~~~java
public class FrostboltLauncher {
    public launch(String target) {
        System.out.println("向" + target + "施放了寒冰箭。");
    }
}
~~~



运行`BeginningMindApplicationTests.test1`，结果如下

> 向Deolin施放了寒冰箭。



很显然，`FrostboltLauncher`是个简单的Bean，简单到直接在类中写一个main方法就足以调试它了。



但是在实际开发中，Deolin无法用main方法来调试一个DAO接口，或是一个Service实现类，这些组件需要整个容器启动起来才能调试到。

这时候，只需要把`FrostboltLauncher`换成其他的Bean，就可以在`@Test`方法里面直接调用它了。

###  调试控制层的请求方法

对于这类方法，Spring + SpringMVC那时候的调试方式是保持项目本地启动，然后用`Postman`软件向服务器发送HTTP请求。

现在，也无需如此了，Spring Boot提供了测试用的Rest模板。

再次改造一下测试启动类

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Log4j2
public class BeginningMindApplicationTests {

    @Autowired
    private TestRestTemplate template;

    @LocalServerPort
    private int port;

    private FrostboltLauncher frostboltLauncher;

    @Test
    public void test1() {
		frostboltLauncher.launch("Deolin");
    }

	@Test
	public void test2() {
        String resp = template.getForObject("http://localhost:" + port + "/demo", String.class);
        assertEquals(resp, "hey, there.");
	}

}
~~~

结果![](/images/7EE.png)

## 项目打包

### jar包

SpringBoot的pom默认packaging标签为jar，所以无需特别的配置，直接运行命令行即可。

~~~shell
$ mvn clean install -Dprod
~~~

> prod代表profile名，如果希望为其他场合打包，更改即可。

打包完毕后，jar文件会出现在target目录下。

### war包

根据[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#build-tool-plugins-maven-packaging)的描述，需要将pom默认packaging标签值改为war，并追加`spring-boot-starter-tomcat`依赖。

~~~xml
	<packaging>war</packaging>
	<!-- 省略其他配置 -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		<!-- 省略其他依赖 -->
	</dependencies>
~~~

更改完毕后，运行命令行即可。

~~~shell
$ mvn clean install -Dprod
~~~

> prod代表profile名，如果希望为其他场合打包，更改即可。

打包完毕后，jar文件会出现在target目录下。

## 后记

新的框架，知识点比较多。

先整理到这里，后续Deolin将会更新Spring Boot在Profiles、参数绑定、校验、持久层、缓存方面的集成方式与实践。