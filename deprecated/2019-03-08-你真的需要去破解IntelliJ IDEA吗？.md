---

title: 你真的需要去破解IntelliJ IDEA吗？

date: 2019-03-08 09:37

updated: 2019-03-08 09:42

tags:
- IntelliJ IDEA

categories: Java

permalink: do-you-really-need-ultimate-idea

---

过时原因：主观性太强，像是一片公众号的文章

## 简介

IntelliJ IDEA除了收费的Ultimate版之外，还有免费且开源的Community版。

这篇POST将会简单介绍一下两者的区别，以及分析是否需要Ultimate

这篇POST将不会教你如何破解IntelliJ IDEA；



## 区别

Ultimate比起Community，多提供以下的功能：

- 语言层面

    1. 对`JavaScript`、`TypeScript`、`CoffeeScript`、`ActionScript`的支持
    2. **能够作为SQL的图形化界面**
    3. 对`CSS`、`LESS`、`Sass`、`Stylus`的支持
    4. 对`XSL`、`XPath`的支持
    5. 对`Ruby`、`JRuby`、`PHP`、`Go`的支持（需要安装对应plugin）

    

- 框架层面
  1. **Spring全家桶**
  2. Java EE
  3. Java Web开发现在各种不流行的框架（指2019年初）
  4. 其他语言各种前端框架
  5. **`Thymeleaf`、`Freemarker`、`Velocity`等模板引擎**

  

- 版本控制

  1. Team Foundation Server
  2. Perforce

  

- 部署环境

  1. **对Tomcat的支持**
  2. 对其他各种Web容器的支持

  

- 构建工具

  1. nodejs NPM
  2. Webpack
  3. Gulp
  4. Grunt

  

- 其他
  1. UML
  2. Dependency Structure Matrix
  3. Detecting Duplicates



> 这些内容来自官方https://www.jetbrains.com/idea/features/editions_comparison_matrix.html
>
> Deolin仅仅对它进行了一些简单的翻译



## 分析

简单来说，如果你用不上Ultimate提供的额外功能，那么你完全没有任何必要使用Ultimate。

以Deolin自己为例：

- Deolin只想用IDEA写好Java，其他语言会选择Visual Studio Code或是更合适的IDE。

- Deolin使用`Sequel Pro`作为SQL图形化界面
- Spring相关插件占用有点高，也无法大幅度提升开发效率
- Deolin身处的项目都没有视图层，数据进出都是JSON，所以各种模板引擎也不必要
- Deolin使用Git做版本控制，有些公司使用SVN
- SpringBoot内置了Servlet容器，IDEA已经不需要支持各种Web容器了
- 目前Java主流的构建工具只有Maven和Gradle



如果你的情况与Deolin类似，那么，使用Community吧



## Community的优势

1. 少了很多功能，代表IDEA的系统占用也会小很多，启动速度也会快不少

2. 不用承担购买Ultimate的开销，或是不用承担破解Ultimate带来的心理负担

   

