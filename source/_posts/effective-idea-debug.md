---
title: IntelliJ IDEA使用技巧 - DEBUG篇

date: 2019-02-06 16:09

updated: 2019-02-06 16:09

tags:
- IntelliJ IDEA

categories: Java

permalink: effective-idea-debug
---

## 简介

这是系列的第三篇，介绍使用IntelliJ IDEA的DEBUG模式调试Java应用时的技巧。



## 查看指定表达式的值

DEBUG模式下，使用`alt+鼠标左键`点击表达式，可以查看点中的表达式的值。

![](/images/effective-idea-debug-01.png)



## 访问某个对象的方法

假如，现在断点停在以下的代码上，现在想要看看users这个List对象的size，

~~~java
userService.createBatch(users);
~~~

那么可以这样操作

![](/images/effective-idea-debug-01.png)



这个`Evaluate Expression`窗口，可以在代码编辑区右键打开，或者你可以去为这个窗口指定一个快捷键（Keymap）



## DEBUG过程中改变某个变量的值

![](/images/effective-idea-debug-03.png)

