---
title: 学习Python（10） 中文乱码问题

date: 2019-04-21 16:40

updated: 2019-04-21 16:40

tags:
- Python

categories: Python

permalink: learning-python-chinese
---

## 简介

这是系列的第10篇，持续收集一些Python中无法正常显示中文时的解决方式



## 正文

### 1

~~~
def chinese(txt):
    if type(txt) is bytes:
        txt = txt.decode('unicode_escape')
    if type(txt) is str:
        txt = txt.encode('latin-1').decode('unicode_escape')
    return txt
~~~

遇到类似于`\u5267\u60c5`的情况，有效

