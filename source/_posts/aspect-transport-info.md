---
title: 切面里的信息向切面外传递
date: 2018-01-07 20:19:00
tags:
- Spring
- AOP
categories: Java
permalink: aspect-transport-info
---



参考djr项目中，AnonUserServiceImpl类的做法

```java
public void insureAnonUserExist() {
        ServletRequestAttributes sra = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
        try {
            current();
        } catch (UnauthorizedException e) {
            // 没有则创建
            String cookieValue = UUID.randomUUID().toString();
            AnonUser anonUser = new AnonUser();
            /*
            	省略
            */
            // 存入cookie
            Cookie cookie = new Cookie(cookieNamePrefix + COOKIE_NAME, cookieValue);
            cookie.setMaxAge(Integer.MAX_VALUE);
            cookie.setPath("/");
            sra.getResponse().addCookie(cookie);
            // 存入request，让后续的this.current()能取到cookie值
            sra.getRequest().setAttribute(cookieNamePrefix + COOKIE_NAME, cookieValue);
        }
    }
```

重点在于`sra.getRequest().setAttribute`，通过Spring提供的类，静态地获取本次请求的request对象，然后把需要传递的信息以setAttribute的方式存入request。
这样一来，即使程序运行到了切面外面，其他的业务层也能通过getAttribute的方式取到信息。