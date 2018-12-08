---
title: Java8 time包的使用总结

date: 2018-07-27 14:25:00

updated: 2018-07-27 14:25:00

tags:
- 总结

categories: Java

permalink: java8-time
---



## 简介

日期`LocalDate`，时间`LocalTime`，日期时间`LocalDateTime`

时区`ZoneId`，时间点`Instant`，时间单位`ChronoUnit`，格式化器`DateTimeFormatter`

 

## 常用API

### LocalDate

1. 当前时间的LocalDate

   ~~~java
   LocalDate localDate = LocalDate.now();
   ~~~

   

2. 指定的LocalDate

   ~~~java
   LocalDate localDate1 = LocalDate.of(2017, 07, 20);
   ~~~

   

3. 加/减 若干个时间单位 

   ~~~java
   // 方式一 （减1个月）
   LocalDate localDate = localDate.plus(-1, ChronoUnit.MONTHS);
   // 方式二 （加1天）
   LocalDate localDate = localDate.plusDays(1);
   ~~~

   

4. 解析日期

   ~~~java
   int dayOfMonth = localDate.getDayOfMonth();
   ~~~

   

5. 是否是润年

   ~~~java
   boolean b = localDate.isLeapYear();
   ~~~

	​	

6. 是否在xxx之前/之后

   ~~~java
   boolean isBefore = localDate1.isBefore(localDate2);
   boolean isAfter = localDate1.isAfter(localDate2);
   ~~~

   

7. 两个日期相差多少个单位

   ~~~java
   long diff = ChronoUnit.DAYS.between(localDate1, localDate2);
   ~~~

   

8. 是否相等

   ~~~java
   boolean b = localDate1.equals(localDate2);
   ~~~



### LocalTime

`LocalTime`大部分的API与`LocalDate`同理，区别之处在于

1. 无法判断是否闰年
2. `LocalDate#toString()`的返回格式为“2017-11-09”，但是，`LocalTime#toString()`的返回格式为“11:16:50.144”（如果毫秒数为0，则最后一段数字不显示），如果想要返回其他格式，则必须借助格式化器。 



### LocalDateTime

`LocalDateTime`大部分的API与`LocalDate`同理，区别之处在于

1. 无法直接判断是否为闰年，需要间接转化一下

   ~~~java
   boolean b = localDateTime.toLocalDate().isLeapYear();
   ~~~

   

2. `LocalDateTime#toString()`的返回格式为“2017-11-09T11:16:50.144”，如果想要返回其他格式，则必须借助格式化器。 



## 类型转化

### 互相转化

1. `LocalDateTime` 转化为 `LocalDate`

   这样转化会失去`LocalDateTime`对象中的“时分秒”信息

   ~~~java
   LocalDate newDate = localDateTime.toLocalDate();
   ~~~

   

2. `LocalDateTime` 转化为 `LocalTime`

   这样转化会失去`LocalDateTime`对象中的“年月日”信息

   ~~~
   LocalTime newTime = localDateTime.toLocalTime();
   ~~~

   

   **可以看到，`LocalDateTime`可以非常简单、易读地转化为`LocalDate`或`LocalTime`，实际上，三个新“时间”类与String、时间戳等互相转化都需要借助`LocalDateTime`作为桥梁。**

   

3. `LocalDate` 转化为 `LocalDateTime`

   这样转化需要提供“时分秒”信息，或是缺省为“0点0分0秒”

   ~~~
   LocalDateTime newDateTime = LocalDateTime.of(localDate, LocalTime.of(0, 0, 0));
   ~~~

   

4. `LocalTime` 转化为 `LocalDateTime`

   这样转化需要提供“年月日”信息，或是缺省为“1970年1月1日”

   ~~~
   LocalDateTime newDateTime = LocalDateTime.of(LocalDate.of(1970, 0, 0), localTime);
   ~~~



### 格式化器

~~~java
DateTimeFormatter formatter1 = DateTimeFormatter.ofPattern(pattern1);
DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern(pattern2);
DateTimeFormatter formatter3 = DateTimeFormatter.ofPattern(pattern3);

// java8 time - String
String content1 = formatter1.format(LocalDate.now());
String content2 = formatter2.format(LocalTime.now());
String content3 = formatter3.format(LocalDateTime.now());

// String -> java8 time
LocalDate localDate = LocalDate.parse(content1, formatter1);
LocalTime localTime = LocalDate.parse(content2, formatter2);
LocalDateTime localDateTime = LocalDate.parse(content3, formatter3);
~~~



### 处理过时的`java.util.Date`

```java
LocalDateTime localDateTime = LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
```



### UNIX时间戳

~~~java
// LocalDateTime -> unix timestamp
long unixTimestamp = localDateTime.atZone(SYSTEM_ZONE).toInstant().getEpochSecond();

// unix timestamp -> LocalDateTime
LocalDateTime date = LocalDateTime.ofInstant(Instant.ofEpochSecond(unixTimeStamp), ZoneId.systemDefault());
~~~



## 与Jackson集成

~~~xml
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-jsr310 -->
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.9.6</version>
</dependency>
~~~



