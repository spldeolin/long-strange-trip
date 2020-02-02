---
title: Javaparser 生成代码时保持原有格式

date: 2020-02-02 14:27

updated: 2020-02-02 14:27

tags:
- Java

categories: Java

permalink: javaparser-original-save
---

## 简介

我们时常会使用Javaparser修改已有的代码，如下所示

~~~java
/* 确保所有的抽象类的类名，都是以“Abstract”作为开头 */
public void transformForAbstract(CompilationUnit compilationUnit) {
    compilationUnit.findAll(ClassOrInterfaceDeclaration.class).stream()
        .filter(c -> !c.isInterface() 
                && c.isAbstract() 
                && !c.getNameAsString.startsWith("Abstract"))
        .forEach(c -> {
            String from = c.getNameAsString();
            String to = "Abstract" + from;
            System.out.println("Renaming class " + from + " into " + to);
            c.setName(to);
        });
}    
~~~



上方的代码节选自Javaparser官网，像这种“修改已有代码”的功能，是Javaparser所能提供的4大基本功能之一，被官方称为`Transform`。

`Transform`完毕后，往往需要将改动点（对于例子而言是`c.setName(to)`）保存磁盘中，如下所示

~~~java
/* transform代码，并将代码以格式化的形式保存到磁盘 */
transformForAbstract(compilationUnit);
compilationUnit.getStorage().ifPresent(Storage::save);
~~~



## `Storage.save`方法的局限性

通过源码可以知道，`CompilationUnit.save`底层是使用`PrettyPrint`对象将`CompilationUnit`对象转化成String类型的代码，再保存到磁盘中的。

如果已有代码的格式是符合规范的，那么没有问题，但如果已有代码存在不合规的格式，比如不统一的缩进方式等，那么所有不“pretty”的地方，都会被强行格式化（code reformat）。

事实上，`transformForAbstract`方法仅仅只想修改抽象类的类名，不想变动其他地方，使用`CompilationUnit.save`作为持久化方案，显然是不适合的。



## LexicalPreservingPrinter

Javaparser还提供了`LexicalPreservingPrinter`，它们能满足 持久化时不格式化代码 的要求。

`LexicalPreservingPrinter`的使用起来也非常简单，只比使用`PrettyPrint`多一个步骤，具体可以在LexicalPreservingPrinter.setup(N)方法的注释上，看到LexicalPreservingPrinter的用法。用法如下所示。

~~~java
/* transform代码，并将代码以保持原格式的形式保存到磁盘 */
LexicalPreservingPrinter.setup(compilationUnit);
transformForAbstract(compilationUnit);
compilationUnit.getStorage().ifPresent(s -> s.save(LexicalPreservingPrinter::print));
~~~



简单来说，就是先`setup`，再`transform`，最后`print`。

`LexicalPreservingPrinter.setup(N)`方法内部会改变compilationUnit的**对象的状态**，即可以提前进行setup，甚至提前到compilationUnit对象被解析出来时（解析`parse`也是Javaparse所能提供的4大功能之一），如下所示

~~~java
StaticJavaParser.getConfiguration().setLexicalPreservationEnabled(true);
CompilationUnit compilationUnit = StaticJavaParser.parse(code);
~~~



## 特别注意

1. `setup`必须在`transform`之前调用，相当于compilationUnit对象需要在transform之前，需要保存自身状态的快照。不这样做的话，`LexicalPreservingPrinter.print`无法返回正确的结果

2. `setup`的开销不可忽视，如果需要解析大量`CompilationUnit`，不建议解析时直接`setLexicalPreservationEnabled(true)`，这样做会增加解析的时间。推荐transform时，确定需要持久化的`CompilationUnit`对象后，再进行`setup`。可以参考如下代码

   ~~~java
   Collection<CompilationUnit> needToSave = new LinkedList<>();
   for (CompilationUnit compilationUnit : veryMany) {
       MutableBoolean hasSetup = new MutableBoolean(false);
       compilationUnit.findAll(ClassOrInterfaceDeclaration.class).stream()
           .filter(this::filterCompilationUnit)
           .forEach(c -> {
               
               // 一个cu下可能有多个class，可能会多次进入这里，但只在第一次进入是保存快照，后续执行到这里时不再覆盖保存快照
               if (hasSetup.isFalse()) {
                   needToSave.add(LexicalPreservingPrinter.setup(compilationUnit));
                   hasSetup.setTrue();
               }
               
               this.setClassDeclarationName()
           });
   }
   needToSave.forEach(cu -> cu.getStorage().ifPresent(s -> s.save(LexicalPreservingPrinter::print));
   
                      
   
   // filterCompilationUnit与setClassDeclarationName方法省略
   ~~~

   