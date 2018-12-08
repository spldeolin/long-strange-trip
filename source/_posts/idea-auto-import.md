---
title: 优化IntelliJ IDEA自动导入的候选项

date: 2018-06-06 08:55:00

updated: 2018-06-06 08:55:00

tags:
- IntelliJ IDEA

categories: Java

permalink: idea-auto-import
---

## 简介

一个Java文件初次使用`Map`时，IDEA会自动引入`java.util.Map`，非常方便。

但是，随着依赖类库的增多，名为`Map`的类会越来越多，IDEA就不知道该引入谁了。

这个时候，IDEA会提示一个`Map`类的全限定名列表，让开发者来决定究竟想要哪个`Map`，如下所示

![](/images/idea-auto-import-1.png)

每次都要手动选择，未免太麻烦了。



## 解决方式

以`Map`、`List`为例，绝大多数情况下，开发者想要的`java.util`包下的这两个类，所以，可以通过追加一些设置，让IDEA能够忽略掉其他包下的`Map`和`List`。



首先，通过Find Action功能打开`Auto Import`

![](/images/idea-auto-import-2.png)



然后，在上图所示的地方追加一些想要排除的类，就可以了。



或者，你可以在`Class to Import`的类上按右方向键，排除它。

![](/images/idea-auto-import-3.png)



第三个方法是直接去IDEA的配置文件加。

`{idea-config-path}/options/editor.codeinsight.xml`

~~~xml
<application>
  <component name="CodeInsightSettings">
    <option name="ADD_IMPORTS_ON_PASTE" value="1" />
    <option name="ADD_UNAMBIGIOUS_IMPORTS_ON_THE_FLY" value="true" />
    <EXCLUDED_PACKAGE NAME="com.alibaba.druid.stat.TableStat.Condition" />
    <EXCLUDED_PACKAGE NAME="com.beust.jcommander.internal.Lists" />
    <EXCLUDED_PACKAGE NAME="com.sun.javafx.collections.MappingChange.Map" />
    <EXCLUDED_PACKAGE NAME="com.sun.mail.imap.Utility.Condition" />
    <EXCLUDED_PACKAGE NAME="com.sun.tools.javac.util.List" />
    <EXCLUDED_PACKAGE NAME="com.sun.xml.internal.xsom.impl.scd.Iterators.Map" />
    <EXCLUDED_PACKAGE NAME="io.undertow.servlet.compat.rewrite.RewriteCond.Condition" />
    <EXCLUDED_PACKAGE NAME="java.util.concurrent.locks.Condition" />
    <EXCLUDED_PACKAGE NAME="org.springframework.context.annotation.Condition" />
    <EXCLUDED_PACKAGE NAME="org.springframework.data.redis.core.index.IndexDefinition.Condition" />
  </component>
  
  <!-- 其他配置省略 -->
~~~

