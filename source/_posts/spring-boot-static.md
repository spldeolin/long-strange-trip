---
title: Spirng Boot处理静态资源

date: 2018-05-05 06:25:00

updated: 2018-05-05 06:25:00

tags:
- Spring Boot

categories: Java

permalink: spring-boot-static
---



## 简介

在Web开发的过程中，作为服务端，往往会碰到需要处理静态资源的情况，大致可以分为两大类——

1. 生成验证码/二维码提供给前端展示，或是生成Excel报表提供给前端下载
2. 将前端上传的文件保存在服务器本地

这篇POST会介绍如何在Spring Boot项目中**优雅地**处理这两种情况。



## 思路

保存文件到服务器本地 -> 将本地路径映射成url -> 将url返回给前端



## 配置

~~~java
@Component
public class WebConfig extends WebMvcConfigurerAdapter {

    /**
     * 本地路径映射
     */
    @Value("${properties.file.mapping}")
    private String mapping;
    
    /**
     * 本地路径
     */
    @Value("${properties.file.location}")
    private String location;
    
	@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        String mapping = properties.getFile().getMapping();
        String location = properties.getFile().getLocation();
        //  addResourceLocations参数必须以/结果，否则识别不到
        if (!StringUtils.endsWith(location, "/")) {
            location += "/";
        }
        registry.addResourceHandler(mapping + "/**").addResourceLocations("file:" + location);
    }

}
~~~

主要是通过引入自定义属性，将本地路径映射成url，这样一下location里的文件，都可以通过

~~~http
http://localhost:${server.port}/${server.context-path}/${properties.file.mapping}/
~~~

为前缀的url访问到。

返回给前端后，根据需要，渲染成一个下载链接，或是将url渲染到img标签等等。



同时`${properties.file.location}`和`${properties.file.mapping}`在后端是可以复用的，

将这两个属性封装到文件上传业务类中，让文件都保存在`${properties.file.location}`下，

然后通过拼接`${properties.file.mapping}`生成文件的映射URL。