---
title: 文档生成器，出世

date: 2018-06-26 15:40:00

updated: 2018-06-26 15:40:00

tags:
- 工具类

categories: Java

permalink: doc-generator
---



## 简介

这篇POST介绍了一个由Java编写，面向Java Web项目的API文档生成器。

生成器是`Cadeau Support`的一个模块。

目前支持的框架有Spring。

基本原理是读取目标项目的Java源文件，解析控制器的JavaDoc，生成文档。

目标项目不需要追加依赖，完全零入侵。

生成器仅要求目标项目的JavaDoc遵循[Oracle的规范](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/javadoc.html)，没有特别的约定，零耦合。

一个具有良好编码习惯的Java开发者，可以通过生成器生成准确的文档，而他自己不需要做任何额外的事情。



## 使用方式

1. 修改项目基础包和控制器所在包的路径
2. 运行`DocGenerator`类的main方法



## 生成示例

我们通过几组图片，直观地展示一下文档生成器所能达到的效果。

### 基本

![](/images/doc-generator-01.png)



### 参数

![](/images/doc-generator-02.png)



### 请求体

![](/images/doc-generator-03.png)



![](/images/doc-generator-04.png)



### 返回值

![](/images/doc-generator-05.png)



![](/images/doc-generator-06.png)



## 相比swagger-ui的优势

swagger-ui最大的优势是与项目本身集成在一起，部署、访问都非常方便，但他的劣势也非常多。

swagger-ui资料较少，难以进行二次开发（识别自定义注解，忽略自定义字段等）。

swagger-ui产生的文档是固定的HTML页面，开发者无法追加一些解释；本生成器产生的文档是md文件，开发者可以追加解释

swagger-ui需要项目依赖swagger-core和swagger-ui，且会对控制层产生较大的代码入侵。

swagger-ui需要开发者编写额外的注解，作为控制器/请求方法/参数/字段的描述，写这些额外注解的时间，足够开发者徒手写个文档了。

每次写过滤器都需要专门放行掉swagger-ui相关的请求，否则文档页面会报错，增加安全模块的复杂度。



## TODO

由于生成的是md文件，那么，这些文档可以部署在hexo上，作为一个文档集合站点。

追加对校验注解的解析。

追加对自定义绑定注解的解析。



## 源码

[spldeolin](https://github.com/spldeolin) / [cadeau-support](https://github.com/spldeolin/cadeau-support)
