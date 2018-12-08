title: Postman使用
date: 2017-12-02 15:01:00

tags: Postman

categories: Java

permalink: try-postman
---

## 简介

Postman是一个可以向服务器发送模拟HTTP请求的软件，用它来调试后端的请求接口非常方便。

## 使用方式

1. 启动web项目

2. 如果是请求的请求体是JSON，需要在`Headers`标签页追加参数`Content-type`，值为`application.json`

3. 在`Body`标签页选择`raw`和`JSON (application/json)`，在下方追加JSON数据

   ![](images/1139226-20171203072442991-1701211193.png)

4. 如果是表单请求，不需要在`Header`标签页追加参数，直接在`Body`标签页选择`x-www-form-urlencoded`，在下方追加Key-Value

   ![](images/1139226-20171203072729272-544484217.png)

5. `Headers`与`Body`编辑完成后，指定请求方式和URL后发送请求。

![](1139226-20171203072916663-1569395345.png)

