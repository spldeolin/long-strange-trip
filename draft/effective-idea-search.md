---
title: Intellij IDEA使用技巧 - 搜索篇

date: 2018-12-12 10:14:00

date: 2018-12-12 10:14:00

tags:
- Intellij IDEA

categories: Java

permalink: effective-idea-search
---





## 简介

新开一个系列，介绍一些IntelliJ IDEA的使用技巧，旨在提升开发人员的编码体验和开发效率。

这些技巧全部整理自Deolin日常的开发过程。



这篇POST是全系列的第一篇，讲的是使用一款IDE时，非常常用的功能——**搜索**



## 事前说明

- 使用的IntelliJ IDEA版本为`2018.3.1`

- 由于每个人的快捷键方案不同，下文中使用到快捷键使用**粗斜体字**表示，全文的最后会给出快捷键对照表

下面开始介绍...



## 根据文件名搜索文件

这个是最日常的行为了，使用**Main menu | Navigate | File...**可以开启搜索框

![](/images/effective-idea-search-01.png)



输入需要的类名，如User就可以搜索到所有的相关类，同时还支持`*`通配符

![](/images/effective-idea-search-02.png)



注意，默认情况下，只支持搜索项目中由开发者编写的文件，无法搜索到Maven依赖，JDK中的源码；如果希望搜索到这些文件，需要勾选`Include non-project files`或是再次按下**Main menu | Navigate | File...**的快捷键

![](/images/effective-idea-search-03.png)



## 根据全限定名定位到一个类

当多个开发者合作开发时，时常会出现A模块开发者需要B模块开发者提供一个API（往往是接口中的方法）。



这种场景下，B开发者可以通过**Copy Reference**功能获取方法的全限定名，然后提供给A开发者，**Copy Reference**功能将会获得如下的全限定名

~~~
com.spldeolin.beginningmind.core.util.Jsons#toBeauty
~~~



A开发者拿到了B开发者发给自己的全限定名，可以使用**Main menu | Navigate | Class...**来定位到目标方法。

![](/images/effective-idea-search-04.png)



单击回车后，IntelliJ IDEA不光会为我们打开这个类，而且会直接将光标定位到方法上

![](/images/effective-idea-search-05.png)



## 全文检索

TODO



## 根据URL检索

TODO



## 定位到文件的具体行

TODO



## 快捷键对照表

|功能名|default快捷键|Eclipse风格的快捷键|
|:---|:---:|:---:|
|Main menu \| Navigate \| File...|||
| Copy Reference                    |               |                     |
| Main menu \| Navigate \| Class... |               |                     |
|                                   |               |                     |
|                                   |               |                     |
|                                   |               |                     |