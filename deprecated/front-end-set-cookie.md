---
title: 前端向HTTP请求中设置Cookie

date: 2017-11-26 09:36

updated: 2017-11-26 09:36

tags: JQuery

categories: 前端

permalink: front-end-set-cookie
---

1. 为HTML引入JQuery Cookie插件

   http://plugins.jquery.com/cookie/

2. JS

   ~~~javascript
   var expiresTime = new Date();
   expiresTime.setTime(expiresTime.getTime() + 0.5 * 60 * 1000);
   $.cookie('token', '曲奇饼干', {
           expires : expiresTime,  // 这里设置cookie过30分钟后失效，如果想指定天数，expiresTime改为数字即可
           path : '/'
       });
   ~~~

3. 服务端

   ~~~java
   /**
        * 请求：测试<br>
        * 取得Http请求中的Cookie
        *
        * @author Deolin
        */
       @RequestMapping(value = "get_cookie", method = RequestMethod.GET)  // 随便写个测试请求，由于path指定了/，所以任何请求中都带有了Cookie。
       public String get_cookie(@CookieValue(value = "token", required = false) String token) {
           if (token == null) {
               return "cookie取不到，超时或未点击“Set Cookie”。";
           }
           return token;
       }
   ~~~

   ​

