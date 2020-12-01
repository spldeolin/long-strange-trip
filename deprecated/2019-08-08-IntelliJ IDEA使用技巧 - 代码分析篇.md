---

title: IntelliJ IDEA使用技巧 - 代码分析篇

date: 2019-08-08 10:40

updated: 2019-08-08 10:40

tags:
- IntelliJ IDEA

categories: Java

permalink: effective-idea-analyze

---

非高深的技巧，但可以考虑发布到公司的语雀，为了技术影响力

## 简介

很久没有更新这个系列了，这次将会介绍一下IntellIJ IDEA的代码分析功能。

这个功能的入口统一在菜单栏的Analyze菜单中



## 分析出所有没有满足某个检查项的代码

例如，现在我希望找出整个项目中，所有从来没有被调用过的本地变量，那么可以那么操作——

1. 双shift并搜索`Run Inspection by Name`这个action

2. 输入`Unused Declaration`，打开这个检查项

3. 在下面的`Members to report`中，仅勾选`Local variables`

   ![](/images/effective-idea-analyze-01.png)

   > 当然，你还可以在`Inspection scope`指定检查的范围

4. IDEA会将分析结果反馈到`Inspection Results`视窗

   ![](/images/effective-idea-analyze-02.png)

   可以看到，分析结果很直观，右侧甚至还提供了4个按钮作为解决方案



## IDEA提供了哪些检查项？

刚才那个例子中，使用到了`Unused Declaration`检查项，而实际上，IDEA内置了非常多的检查项。

可以通过`Preferences | Editor | Inspections`，查看它们各自的名字和具体描述

![](/images/effective-idea-analyze-03.png)