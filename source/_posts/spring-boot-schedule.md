---
title: Spring Boot启用定时任务

date: 2018-05-12 16:20:00

tags:
- Spring Boot

categories: Java

permalink: spring-boot-schedule
---

## 简介

Spring Boot直接支持定时任务，无需集成，直接启用即可



## 配置类

~~~java
@EnableScheduling
public class ScheduleConfig {
}
~~~



## 任务类

~~~java
@Component
@Log4j2
public class oneTask {

	@Scheduled(fixedDelay = 1000)
	public void send() {
		log.info("+1s");
	}

}
~~~



这里演示的是最简单的固定间隔的定时任务。

如果有跟复杂的定时需求，可以使用`@Scheduled`的`cron`属性，常用的cron表达式有

- 每天0点        0 0 0 * * ?
- 每周一0点    0 0 0 ? * MON


- 每月1日0点  0 0 0 1 * ?



更多写法可以[参考这里](http://www.pdtools.net/tools/becron.jsp)。

