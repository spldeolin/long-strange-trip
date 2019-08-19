---


title: 学习Python（12） “时间”

date: 2019-08-19 16:39

updated: 2019-08-19 16:39

tags:
- Python

categories: Python

permalink: learning-python-time

---

## 简介

“时间”类的数据类型，一般在高级编程语言中都是内置的，Python也不例外。

系列的第12篇将会介绍Python中的两个时间类库——`time`和`datetime`



## time

1. time.time() 读取1970-01-01 00:00:00至当前系统时间的秒数，返回值是个浮点数，即包含了毫秒数、尾秒数等信息
   - 存在2038限制
2. time.sleep() 可以理解为Java中的Thread.sleep()



## datetime.datetime

1. datetime是库名，库内有一个class也叫datetime，即datetime数据类型，代表日期时间
2. 构造datetime
   - dt = datetime.datetime.now()
   - dt = datetime.datetime(2012, 12, 31, 0, 0, 0)
     - 可以看一眼构造方法，datetime精确到微秒，还支持时区信息
   - dt = datetime.datetime.fromtimestamp(time.time())
3. 解析datetime
   - Year = dt.year
   - day = dt.day
   - ...etc
4. datatime支持`<`、`>`、`==`、`!=`的比较



## datetime.timedelta

1. datetime库中还有一个class是timedelta，代表时间的变化量
2. datetime.timedelta.total_seconds() 返回变化量折合的秒数

3. timedelta可以直接与datetime相加，两个datetime相减直接返回timedelta类型。**Python的类型转换真是太灵活了**



## 格式化

1. dt.strftime('%Y-%m-%d %H:%M:%S')

2. string -> datetime的函数是strftime