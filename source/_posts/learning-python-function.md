---
title: 学习Python（3） 函数

date: 2019-03-31 13:29

updated: 2019-03-31 13:29

tags:
- Python

categories: Python

permalink: learning-python-function
---

## 简介

这是系列的第3篇，这篇的示例代码比较多，所以不用思维导图了



## 正文

### 定义、使用

没有大括号，只有冒号

```python
def do_something(what_thing):
    print(what_thing + ' done.')

do_something('writing')
```



### 关键字参数

```python
def someone_do_something(who, what_thing):
    print(what_thing + ' done by ' + who + ".")

someone_do_something(who = 'Deolin', what_thing = 'writing')
```



### 参数默认值

```python
def do_you_like_deolin(y_n=True):   # 等号两端不要有空格
    if y_n:
        print('yes, i do')
    else:
        print('no, i do not')


do_you_like_deolin()
```



### 函数修改列表

```python
def modify(list, str):
    list[0] = str

raw = list(range(0,10))

modify(raw, 'first')
print(raw)

modify(raw[:], 'second')         # 注意[:]
print(raw)
```



### 不定参数

```python
def mmm(*args):
    for v in vs:
        print(v)
mmm(1,2,3,4,'a')
```

在这里，values是个元组



### 不定关键字参数

```python
def iii(**kwargs):
    for k, v in kvs.items():
        print(k + '---' + str(v) + '\n')

iii(w='1', a='b', b=1) # 这里的key不能是数字，也不能以数字开头
```



### 什么是模块

定义了若干`函数`的的`py文件`，被称为`模块`



### 导入整个模块

```python
import pizza as p

p.greeting()
p.serve()
p.clean()
```

`import`后跟的是py文件名



### 导入模块里的某个函数

```
from pizzz import greeting, clean

greeting()
clean()
```





