---

title: Java8 time包指南

date: 2019-06-06 22:25

updated: 2019-06-07 22:50

tags:
- Java

categories: Java

permalink: java8-time-guide

---

## 简介

这次，Deolin将会全面整理一份关于Java8新的time包的使用指南。

在此之前，对time包做过两次简单的整理，在这篇POST出现后，它们将会被废弃。

[过时1](https://github.com/spldeolin/long-strange-trip/blob/master/deprecated/java8-time.md) [过时2](https://github.com/spldeolin/long-strange-trip/blob/master/deprecated/java8-time-ex.md)



## 新的类型

### LocalDate与LocalTime

- 类如其名，它们分别代表“年月日”这样的**日期**与“时分秒”这样的**时间**

- 这两个类型都实现了`Temporal`接口

- 构建对象

  ~~~java
  // 方式一：通过API
  LocalTime localTime = LocalTime.of(21, 1, 1);
  LocalDate localDate = LocalDate.of(2012, 12, 31);
  LocalTime localTime2 = LocalTime.now();
  LocalDate localDate2 = LocalDate.now();
  
  // 方式二：通过转化字符串
  LocalTime localTime3 = LocalTime.parse("12:12:12", dateTimeFormatter);
  ~~~

- 解析时间

  ~~~java
  // 方式一：通过API
  int year = localDate.getYear();
  int hour = localTime.getHour();
  // ...
  
  // 方式二：通过TemporalField
  int month = localDate.get(ChronoField.MONTH_OF_YEAR); // ChronoField枚举实现了TemporalField
  int minute = localTime.get(ChronoField.MINUTE_OF_HOUR);
  // ...
  ~~~
  
  示例中提到的`TemporalField`接口值得被关注，它有许多实现类，这意味着Java提供了许多种解析时间的方式，例如
  
  - get(IsoFields.DAY_OF_QUARTER) 返回该时间是所在季度的第几天
  - get(ChronoField.MINUTE_OF_HOUR) 返回该时间的分钟数
  
  多查`TemporalField`的实现类，并善用它们，可以让原本复杂的时间解析变得非常简单。



### LocalDateTime

- 这个类型是`LocalDate`与`LocalTime`的合体，它也实现了`Temporal`接口

- 构建对象

  ~~~java
  LocalDateTime ldt1 = LocalDateTime.of(2000, 1, 1, 12, 1, 1);
  LocalDateTime ldt2 = LocalDateTime.of(localDate, localTime);
  LocalDateTime ldt3 = localDate.atTime(12, 1, 1);
  LocalDateTime ldt4 = localDate.atTime(localTime);
  LocalDateTime ldt5 = localTime.atDate(localDate); 
  ~~~

  不得不说，真的设计地非常灵活。

- `LocalDateTime`的API与`LocalDate`和`LocalTime`很类似



### Instant

- `Instant`是面向计算机的时间类型，它也实现了`Temporal`接口
- 一般通过秒和纳秒来构建`Instant`对象
- `Instant`不支持读取值，调用get方法会抛出异常



### Duration与Period

- 它们分别表示两个`Temporal`之间的时间间隔

- 这两个类都没有实现`Temporal`接口

- 构建对象

  ~~~java
  // Duration表示LocalDate以外的两个Temporal之间的间隔
  Duration d = Duration.between(localTime, localTime2);
  Duration d2 = Duration.between(localDateTime, localDateTime2);
  Duration d3 = Duration.between(instant, instant2);
  
  // Period表示两个LocalDate之间的间隔
  Period period = Period.between(LocalDate.now(), LocalDate.of(2019, 12, 31));
  ~~~

- 解析**时间间隔**

  `Duration`与`Period`对象都支持使用`get`方法进行解析，这个方法支持所有`TemporalUnit`的实现类

  

## 操作Temporal

这里以`LocalDate`为例，介绍一下如何操作`Temporal`对象

- 直接修改值

  ~~~java
  // 方式一：使用API
  LocalDate date = LocalDate.of(2014, 12, 1); // 2014-12-01
  LocalDate newDate = date.withYear(2018); // 2018-12-01
  
  // 方式二：使用TemporalAdjuster（也可以使用自定义的TemporalAdjuster实现类）
  LocalDate newDate2 = date.with(TemporalAdjuster.lastDayOfMonth()); // 2014-12-31
  ~~~

  注意，`LocalDate`是不可变对象

- 相对修改值

  ~~~java
  LocalDate date = LocalDate.of(2014, 12, 1); // 2014-12-01
  LocalDate newDate = date.plusYears(1).minus(1, ChronoUnit.MONTHS); // 2015-11-01
  ~~~



## 转换为String

- 会用到一个类——`DateTimeFormatter`



## 时区问题

- 新的时区类型为`ZoneId`和`ZoneOffset`，其中`ZoneOffset`是`ZoneId`的子类。

- `LocalDate`、`LocalTime`、`LocalDateTime`这三个类型都是不带时区信息的，当他们与`ZoneId`结合后，会生成`ZonedDateTime`

  ~~~java
  LocalDate localDateTime = LocalDateTime.of(2019, 1, 2, 12, 12, 12);
  // 通过ZoneId对象
  ZonedDateTime zonedDateTime1 = localDateTime.atZone(ZoneId.of("Europe/Rome"));
  
  // 通过ZoneOffset对象
  ZonedDateTime zonedDateTime2 = localDateTime.atZone(ZoneOffset.of("-05:00"));
  
  ~~~