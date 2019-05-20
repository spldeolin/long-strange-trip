---

title: Python常用的字符串检查

date: 2019-05-20 17:14

updated: 2019-05-20 17:14

tags:
- Python

categories: Python

permalink: python-string-check

---

 ## 简介

基本所有有字符串类型的编程语言中，对字符串进行操作或是检查的代码都是最多的。

这篇POST将会以类比的方式，介绍一下Python常用字符串检查的方式



## 判断字符串是否为空/不为空/为空白/不为空白……

Java版

~~~java
b = StringUtils.isEmpty(s);
b = StringUtils.isNotEmpty(s);
b = StringUtils.isBlank(s);
b = StringUtils.isNotBlank(s);
~~~



Python版

~~~python
b = not s
b = bool(s)
b = not s.strip() # 无参的`strip()`会清除字符串首尾所有的空格、换行符、制表符
b = bool(s.strip())
~~~



## 判断字符串是否包含了列举出的子字符串中的一个

Java版

~~~java
if (StringUtils.containsAny(aaa, "q", "w", "e"))
~~~



Python

~~~python
if any(sub in aaa for sub in ('q', 'w', 'e')):
~~~



`any`函数会去遍历参数，这个参数必须是可迭代的，一旦`any`遍历到为`True`的元素，就返回`True`
`any`里面的表达式叫做`生成器表达式`，是`生成器函数`的简便写法（可以理解称Java中`users.forEach(log::info)`这样的写法）

示例中的表达式转化成生成器函数是这样的

~~~python
def g1():
    for sub in ('q', 'w', 'e'):
        yield sub in aaa 
if (any(g in g1()))
~~~

无论是生成器表达式还是生成器函数，他们都代表一个生成器对象，生成器对象是可迭代的



## 判断字符串是否包含了所有的列举出的子字符串

Java版

~~~java
if (aaa.contains("q") && aaa.contains("w") && aaa.contains("e")) // StringUtils里面没有containsAll这样的功能
~~~



Python版

~~~python
if all(sub in aaa for sub in ('q', 'w', 'e'))
~~~



理解了生产器表达式以后，很多写法就很好理解了



## 判断一批字符串是否存在至少一个空字符串

Java版

~~~java
if (StringUtils.isAnyEmpty(aaa, bbb, ccc, ddd))
~~~



Python版

~~~python
if any(not sub for sub in (aaa, bbb, ccc, ddd)):
~~~



## 判断一批字符串是否没有空字符串

Java版

~~~java
if (StringUtils.isNoneEmpty(aaa, bbb, ccc, ddd))
~~~



Python版

~~~python
if all(bool(sub) for sub in (aaa, bbb, ccc, ddd))
~~~

