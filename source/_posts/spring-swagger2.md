---
title: Spring集成Swagger2

date: 2018-02-28 20:12

updated: 2018-02-28 20:12

tags:
- Spring
- Swagger

categories: Java

permalink: spring-swagger2
---

## 简介

Spring项目集成了Swagger之后，前端开发者可以通过`/swagger-ui.html`请求，**可视化**地获取到服务端开发者提供的接口文档；对于服务端开发者来说，控制层的各种路由和参数绑定注解会被Swagger解析成文档，无需开发者去写文档。

Swagger极大地降低了双方开发者交流的成本。



## 追加依赖

```xml
<!-- Swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```



## Spring MVC

如果项目是Spring MVC，那需要确保`dispatcherservlet-config.xml`中声明了

~~~xml
<mvc:default-servlet-handler/>
~~~



## 配置

```java
@Configuration
@EnableSwagger2
public class Swagger2Configuration {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .enable(true)    // 启用/禁用开关
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.spldeolin.beginningmind"))
                .paths(PathSelectors.any())
                .build();
    }

}
```

`@Bean`是非必须的。

`@Configuration`是**必须**的，哪怕类里面没有声明`@Bean`，也是必须的，否则swagger页面会404。



## 使用Swagger2

- 控制器

  ~~~java
  @Api(tags = "汉字")
  ~~~

- 请求方法

  ~~~java
  @ApiOperation("汉字2")
  ~~~

- RequestParam或PathVariable参数

  ~~~java
  @ApiParam("简体汉字")
  ~~~

- 请求Body

  ~~~java
  @ApiModelProperty("汉字解释")
  ~~~



## Spring Boot下根据profiles开启Swagger

> 2018/05/19 11:13

如简介所说，Swagger基本只在开发阶段有用，测试阶段基本用不到，至于生产环境，显然是不应该启用Swagger，一旦启动，等于把服务端请求接口信息全部暴露出去了。

比较理想的应对方式是根据profile来决定Swagger是否启用，改造一下`SwaggerConfig`即可。

~~~java
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
            .enable(enable())    // 启用/禁用开关
            .apiInfo(apiInfo())
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.spldeolin.beginningmind"))
            .paths(PathSelectors.any())
            .build();
    }

    @Autowired
    private Environment environment;

    /**
     * @return 生产环境返回false，其他环境返回true
     */
    private boolean enable() {
        for (String activeProfile : environment.getActiveProfiles()) {
            if ("prod".equals(activeProfile)) {
                return false;
            }
        }
        return true;
    }
~~~



Swagger被禁用的情况下会显示这样的画面

![](/images/spring-swagger2.png)

比较萌。