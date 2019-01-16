---
title: Ajax与Spring的跨域问题

date: 2018-05-03 20:06

updated: 2018-05-03 20:06

tags:
- Spring

categories: Java

permalink: cors
---

## 简介

这个POST介绍了如何解决类似`No 'Access-Control-Allow-Origin' header is present on the requested resourc`的跨域错误。



## 服务端

~~~java
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**").allowedOrigins("*")
                        .allowedMethods("*").allowedHeaders("*");
            }
        };
    }
~~~



## 前端

~~~javascript
$.ajax({
    
    // ...
    
    dataType : "json",
    contentType : 'application/json',
    
    crossDomain:true,						// 如果涉及到跨域Cookie，那么这项是必须的
    xhrFields: {  withCredentials: true  },   // 如果涉及到跨域Cookie，那么这项也是必须的
    
    // ...
    
    success : // ...
    error : // ...
});
}
~~~

