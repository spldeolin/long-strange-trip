---

title: 解决使用Javaparser解析大量项目产生的OOM问题

excerpt: 体量大了，直觉上不会发生的问题会逐渐暴露

date: 2020-05-05 08:46

updated: 2020-05-05 08:46

tags:
- Javaparser

categories: Java

permalink: allison1875-oom

---

## 简介

最近，Allison1875碰到了一个问题，它在为1个分布式系统处理源码时，出现了OOM的情况，这个分布式系统的源码有60多个project，其下有共340多个module。

在这样的量级下，4G的内存完全不够用，而Allison1875作为一个在本地开发环境使用的源码工具，也能难为其分配更大的内存，也不应该因为目标项目增加而无限制地为其分配更大的内存。这样的限制是符合直觉的，显然使用IDE同时打开60+多个project也是困难的。



## 问题所在

原先，Allison1875解析一个项目的流程是这样的

1. 使用`CollectionStrategy`对象（使用哪种实现取决与是否开启类加载），收集到项目路径下的所有`SourceRoot`
   - 关于类加载， 如果开启了类加载，`CollectionStrategy`会选用`SymbolSolverCollectionStrategy`作为实现。这需要用户先对项目进行打包，再使用Allison1875，Allison1875运行后会先解压这个jar文件，然后获取jar内包含的classpath和每个lib的路径，构造出一个`URLClassLoader`对象用于先后构造`ClassLoaderTypeSolver`和`SymbolSolverCollectionStrategy`对象
2. 对每个`SourceRoot`调用`tryToParse`方法，收集到每个SourceRoot下的所有`CompilationUnit`（`CU`）
3. 所有`CU`保存到`AstContainer`中，后者还提供了许多API用于根据全限定名查询`CU`下的各种AST节点

这个方案最大的优点是将整片AST森林直接加载进了内存，只需要全项目解析一次即可，节省时间。

这个方案最大的缺点是，没有考虑到一旦项目多了，大量CU堆积，每个CU下因为词法解析而产生的Range和Token对象会让内存受不了，事实上这两类对象是OOM时堆内占比最大的类型

这个方案在类加载方面还存在一粗糙的地方。项目之间，甚至module之间都用可能依赖不同版本的lib，不应该全部收集在起来构造一个巨大的ClassLoader，一旦项目多了，一定会出现版本问题。



## 解决思路

Deolin在最近几天已经重构了Allison1875的base模块的collection包，基本解决了OOM问题，这篇POST会讲一下重构的思路和几个比较关键的要点

- 使用游标方式代替加载整个森林的方式。游标对象每次调用`next`只能获取一个`SourceRoot`下的下一个CU，如果这个`SourceRoot`下所有`CU`都被迭代过了，调用下一个`SourceRoot`的`tryToParse`方法，释放对上一个`SourceRoot`和其下`CU`的引用，即确保Ast游标只保存一个SourceRoot下的所有`CU`，如果这还能OOM，那么这个项目平时是怎么使用IDE进行开发的？

  **注意这里一定要释放掉上一个`SourceRoot`，因为`SourceRoot`对象下的cache属性，持有的所有解析出的`CU`，不释放的话，CU依然会在堆内堆积**

- 关于类加载。这次改造成了为每个SourceRoot都构建一个ClassLoader，因为一个pom文件内不可能依赖多个版本的同个lib。新的ClassLoaderFactory的实现被设计成了通过classpath和lib.jar的path来构造ClassLoader对象的形式，因为classpath除非项目移动位置了，否则不会变，lib.jar的path除非pom.xml更新了，否则也不会变。这样设计的好处是Allison1875启用类加载时不需要解压jar包了，也不需要用户手动打包了。

- 通过一个新的Tool来生成classpath和lib.jar，上一点提到了使用classpath和lib.jar两个信息，但是要让用户去统计每个project的每个module的这两个信息显然是不合适的，虽然两者基本不太会变动。更合理的方式是通过一个Tool，来执行`dependency:copy-dependencies`和`clean compile`两个命令，产生classpath和lib.jar的path两个信息，然后制成yaml配置的片段，提供给base-config.yaml

- 原先的`AstContainer`的机制依然可以保留，不过现在只保存一个SourceRoot下的AST节点，可能没什么必要了



## 后记

这篇POST只是描述了几个要点，所有的重构工作已经完成，反映在Github项目仓库中了。

这个问题的产生，真正地印证了两句话

> 数据量/并发量 大了，将会面对许多之前直觉上觉得不会产生的问题。

> 定位到OOM的源头后，更大的挑战是定位源头对象在什么地方没有及时释放引用。