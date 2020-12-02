---

title: MacOS Excel CSV文件乱码问题

excerpt: 可能真的不是你的错

date: 2020-02-28 16:26

updated: 2020-02-28 16:26

tags:
- Java
- Mac OS

categories: Java

permalink: macos-excel-csv

---

## 简介

在Mac OS操作系统使用Microsoft Excel打开一个CSV文件时，里面的中文变成了生僻字乱码，是一个常见的正常现象。



## 环境层面存在的问题

在Mac OS，Microsoft Excel使用的是`GBK`编码，这是个客观的事实。（finder的空格预览功能使用的是`UTF-8`）

换句话说，代码将逗号分割的字符串，按`UTF-8`的编码，输出到一个csv文件中，那么在Mac OS使用Microsoft Excel打开后，中文一定是乱码。你的代码没有问题，你理应使用`UTF-8`编码输出。



## 解决方案

在开发**导出CVS**这类的功能时，你很难确定用户所处的操作系统，更难确定用户用什么软件来打开。

可以考虑生成两份文件打包成`zip`文件导出，一份用`UTF-8`编码，一份用`GBK`，以作兼容。