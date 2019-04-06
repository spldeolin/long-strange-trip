---
title: 学习Python（7） 命名规约

date: 2019-04-06 17:01

updated: 2019-04-06 17:01

tags:
- Python

categories: Python

permalink: learning-python-name
---

## 简介

这是系列的第7篇，主要记录一下Python世界中那些约定俗成的命名规约。



## 正文

### 驼峰还是蛇形？

在Python的官方的示例代码中，出现过3种命名法

- `SCREAMING_SNAKE_CASE`：常量
- `PascalCase`：类名
- `snake_case`：变量名、函数名、属性名、方法名、包名、模块名、py文件的文件名

与官方保持一致即可。



### 下划线的作用

1. 单前导下划线 （如`_var`）

   代表这个属性或方法是受保护的，能够访问但不建议访问。

   “不去管它就好”

2. 单末尾下划线 （如`def_`）

   用于解决变量名被Python关键字占用的情况

3. 双前导下划线 （如 `__var`）

   代表不能访问，dir()中找不到它，因为它被Python修饰成别的方法名了

4. 双前导和末尾下划线 （如`__var__`）

   代表内置属性，可以访问，但绝对不要创造这样命名的属性或方法

5. 单下划线

   用来为不需要被引用变量来占位，常见用法有以下两种

   ~~~python
   for _ in range(5): # 作用类似Java中的for-index循环
       print('hh')
   ~~~

   ~~~python
   user_profile = (1, 'Deolin', 'Hangzhou', 'male', ['Java', 'Python'])
   _, name, _, _, langs = user_profile
   print(name)
   print(langs)
   ~~~

   