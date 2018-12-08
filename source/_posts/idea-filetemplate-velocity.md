---
title: IntelliJ IDEA文件模板与Velocity语法

date: 2018-05-16 17:21:00

tags:
- IntelliJ IDEA
- Velocity

categories: Java

permalink: idea-filetemplate-velocity
---

## 简介

这篇POST将会介绍IntelliJ IDEA中的`File Templates`功能，与一种模板语法——Apache的`Velocity`。

这个功能可以大幅度提高开发效率。



## File Template

### 什么是File Template

下面这张图能够最大程度上介绍什么是File Templates——

![](/images/idea-filetemplate-velocity-1.gif)



### 创建File Template

可以在`Settings...` 中找到关于File Template的设置

![](/images/idea-filetemplate-velocity-2.png)



只要有足够的创意，就可以创造出大量适合自己的File Template。



## 动态模板

IDEA在File Template功能中预置了一些占位符，用于获取环境信息，从而设计动态模板。

这里列举一些常用的——

- `${NAME}` 代表这里填入的值

  ![](/images/idea-filetemplate-velocity-3.png)

- `${USER}` 代表操作系统当前登录用户

- 各种当前时间

  - `${YEAR}`  e.g.: `2018`
  - `${MONTH}` e.g.: `05`
  - `${DAY}` e.g.: `01`
  - `${MONTH_NAME_SHORT}` e.g.: `Jan`
  - `${MONTH_NAME_FULL}` e.g.: `February`
  - `${DAY_NAME_SHORT}` e.g.: `Tue`
  - `${DAY_NAME_FULL}` e.g.: `Monday`
  - `${HOUR}` e.g.: `08`
  - `${MINUTE}` e.g.: `36`

- `${PROJECT_NAME}` 当前项目名



## 进阶动态模板

预置的占位符可能依然无法完全满足复杂的需求，这时候要借助`Velocity`语法了，File Template默认支持这种语法。

Deolin不会在这篇POST详细介绍`Velocity`，而是列举出一些写File Template常见的需求和应对方式。



1. 无论开发者输入了什么类名，首字母都自动转大写

   ~~~velocity
   #set ($validName = $NAME.substring(0, 1).toUpperCase() + $NAME.substring(1))
   public class ${validName} implements Serializable {
   
   }
   ~~~

   

2. 让开发者在创建Java文件时就确定类级注释

   ~~~velocity
   /**
   #if ($Description != "") * $Description
    *
   #end * @author $USER $YEAR/$MONTH/$DAY
    */
    public class ${NAME} {
        
    }
   ~~~

   

3. 如果用户填写类名不是以”DTO“、“VO”结尾的，那就缺省“DTO”结尾。

   ~~~velocity
   ## 确保以NAME以DTO、Input、PO或VO结尾，缺省为以DTO结尾
   #if (!$validName.endsWith("DTO")
           && !$validName.endsWith("Input")
           && !$validName.endsWith("PO")
           && !$validName.endsWith("VO"))
       #set ($validName = $validName + "DTO")
   #else
       #set ($validName = $validName)
   #end
   ~~~

   

4. package语句

   ~~~velocity
   #if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
   ~~~

   

5. 驼峰字符串转蛇形字符串

   ~~~velocity
   ## Model名称转化为snake分割（将所有大写字符前面加下划线，然后字符串转小写，最后删除第一个下划线）
   #set( $regex = "([A-Z])")
   #set( $replacement = "_$1")
   #set( $classMapping = $modelName.replaceAll($regex, $replacement).toLowerCase().substring(1))
   ~~~

   

## 成品

Deolin这里提供了自己日常所用的`File Template`，仅供参考。

https://github.com/spldeolin/intellij-idea-config/tree/master/config/fileTemplates