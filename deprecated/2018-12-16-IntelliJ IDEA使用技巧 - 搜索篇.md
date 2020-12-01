---
title: IntelliJ IDEA使用技巧 - 搜索篇

date: 2018-12-16 08:01

updated: 2018-12-16 09:30

tags:
- IntelliJ IDEA

categories: Java

permalink: effective-idea-search
---

非高深的技巧，但可以考虑发布到公司的语雀，提高技术影响力



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

IDEA提供了全文检索了，可以通过**Find in path...**打开全文检索界面，



![](/images/effective-idea-search-06.png)

可以看到，全文检索功能的订制性比较强。

一般输入关键字以后，`Preview`就会显示检索结果了

![](/images/effective-idea-search-07.png)





## 根据URI检索

作为一个服务端开发者，与客户端或是前端联调时，如果出现BUG，对方往往会趋向于直接给你 报错的URL，比如`/user/create`。

如果是简短的URI，还能通过`UserController#create()`这样的规律去遇到到控制层，但如果是类似于`/store/order/refundOrder/detail/10001`这样的长URI呢？



IDEA本身不提供根据URI检索请求方法，但是我们可以借助`RestfulToolkit`来帮助我们。

安装完该插件后，使用**Main menu | Navigate | Service...**便可打开界面

![](/images/effective-idea-search-08.png)



输入URI，就能定位到具体的方法了

![](/images/effective-idea-search-09.png)



注意，这里的输入的内容也是支持`*`通配符的



## 定位到文件的具体行

一般项目中都会有日志打印，而打印出来的每一条日志，一般都会带上行号。

现在我们已经通过上文提到的各种方法，定位到具体类的具体方法了，有没有办法直接定位到行呢？

实际上IDEA为我们提供了这样的功能——**Main menu | Navigate | Line/Column... ** 在打开的对话框中直接输入行号或是行号:列号即可



## 快捷键对照表

|功能名|default快捷键|Eclipse风格的快捷键|
|:---|:---:|:---:|
|Main menu - Navigate - File...|ctrl + shift + N|ctrl + shift + R|
| Copy Reference                    | ctrl + alt + shift + C | ctrl + alt + shift + C |
| Main menu - Navigate - Class... | ctrl + N | ctrl + shift + T |
| Find in Path...                      | ctrl + shift + F | ctrl + H |
| Main menu - Navigate - Service... | ctrl + \ 或是 ctrl + alt + N | ctrl + \ 或是 ctrl + alt + N |
| Main menu - Navigate - Line/Column | ctrl + G | ctrl + L |



如果你有自己的一套快捷键策略，你可以打开`File | Settings | Keymap`，通过上表中的`功能名`来检索到当前策略下，该功能的快捷键