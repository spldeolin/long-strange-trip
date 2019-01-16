---
title: Java8 time包的使用总结 补充

date: 2018-08-23 15:37

updated: 2018-08-23 15:37

tags:
- 总结

categories: Java

permalink: java8-time-ex
---



## 计算日期属于所在年份的第几周

~~~java
WeekFields weekFields = WeekFields.of(Locale.getDefault());
localDate.get(weekFields.weekOfWeekBasedYear());
~~~



## 指定日期所在月的第一天，最后一天

~~~java
LocalDate result = LocalDate.of(2018, 08, 23).with(TemporalAdjusters.firstDayOfMonth());

result = LocalDate.of(2018, 08, 23).with(TemporalAdjusters.lastDayOfMonth());
~~~



 ## 指定日期所在周的周一（周*）

~~~java
LocalDate result = LocalDate.of(2018, 08, 23).with(DayOfWeek.MONDAY);

// ...

result = LocalDate.of(2018, 08, 23).with(DayOfWeek.SUNDAY);
~~~







