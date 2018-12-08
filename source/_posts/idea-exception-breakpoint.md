---
title: 异常断点，空指针追猎者

date: 2018-06-04 18:17:00

updated: 2018-06-04 18:17:00

tags:
- IntelliJ IDEA

categories: Java

permalink: idea-exception-breakpoint
---

## 简介

这篇POST主要介绍了IntelliJ IDEA的“异常断点”功能。

这个功能对Java来说，简直可以称之为**空指针追猎者**了。



##  设置异常断点

首先用`Find Action...`找到并打开`View Breakpoints`

![](/images/idea-exception-breakpoint-1.png)



左侧的`Java Exception Breakpoints`下默认只有`Any exception`，在这里可以追加一些具体的异常类型，被勾选的异常，会在程序抛出该异常前，停留在抛出异常的代码上。



以空指针为例（毕竟它是最难不借助stacktrack定位到案发现场的异常了），我们追加一个空指针异常断点。

![](/images/idea-exception-breakpoint-2.png)



## 示例

写一个简单的请求方法

~~~java
    @GetMapping("/canIBeNull")
    Object get() {
        Integer s ;
        if (new Random().nextBoolean()) {
            s = null;
        } else {
            s = 1;
        }
        return s.toString();
    }
~~~



有时候会这样...

![](/images/idea-exception-breakpoint-3.png)

而有时候会这样...

![](/images/idea-exception-breakpoint-4.png)

