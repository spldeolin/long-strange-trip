---
title: 利用Spring监听器打印每次请求的执行时间

date: 2018-04-26 16:02

updated: 2018-04-26 16:02

tags:
- Spring
- AOP

categories: Java

permalink: processing-time-log-listener
---

## 简介

这篇POST介绍了借助`ApplicationListener`接口派生一个HTTP请求的监听器，用于打印每次请求的执行时间。



## 本地

~~~java
/**
 * HTTP请求事件监听：打印每个请求执行时间的日志（毫秒）
 *
 * @author Deolin
 */
@Component
@Log4j2
public class ServletRequestProcessingTimeLogListener implements
        ApplicationListener<ServletRequestHandledEvent> {

    private static final String SEP = "\t";

    @Override
    public void onApplicationEvent(ServletRequestHandledEvent event) {
        log.info("[" + event.getMethod() + "]" + event.getRequestUrl() + SEP +
                event.getProcessingTimeMillis());
    }

}
~~~



## 效果

![](/images/processing-time-log-listener-01.png)

![](/images/processing-time-log-listener-02.png)

