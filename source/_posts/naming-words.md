---

title: 个人向命名用词

date: 2020-03-28 13:37

updated: 2020-04-10 13:24

tags:
- Java

categories: Java

permalink: naming-words

---

## 简介

用于命名的单词一览，个人向，持续更新



## 单词一览

- `get`  `set`

  不使用这2个单词，因为POJO的Setter和Getter已经足够多了

- `obtain`  `find`

  获取（都表示获取一个或多个）

- `isXxx`  `xxxOrNot`  `hasXxx`  `canXxx`

  用于布尔类型的属性、变量或是返回布尔类型的短方法

- `estimate`

  判断（用于返回布尔类型的长方法）

- `ensure`

  确保某件事，如果不满足则抛出异常

- `xxxMight`

  方法内部可能什么都不做

- `build`

  new一个对象，初始化后返回

- `from`

  构造一个对象，参数是一个DTO，内部可以包含了若干次setXx(getXx())

- `of`

  构造一个对象，参数有多个，内部可以包含若干次setXx(...)

- `begin`、`end`

  开始、结束（start听上去更像是在操作线程，所以不使用）

- `save`

  落到磁盘（指内部进行了一次或若干次insert、update、delete）

- `create`

  以insert为主的save

- `extract`

  提取，指list.stream().map().collect()

- `collectIntoMap`

  指List→Map

- `increase`、`decrease`

  增减

- `increment`、`decrement`

  增减量，均为正数
  
- `result`

  在方法内部声明用于表示返回值，最后一次调用一定是return result;