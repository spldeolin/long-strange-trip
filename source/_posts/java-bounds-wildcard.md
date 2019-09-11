---

title: Java泛型中的上、下界通配符

date: 2019-09-11 14:09

updated: 2019-09-11 14:09

tags:
- Java

categories: Java

permalink: java-bounds-wildcard

---

## 简介

这两天在回顾《Thinking in Java》的泛型，不得不说每次回顾这本书总会有新的收获，

这次终于搞懂了`List<? extends Fruit>`和`List<? super Apple>`这两种通配符了，现在Deolin尝试一下能不能解释清楚它们。



## 准备工作

解释之前，首先定义几个空的Class

~~~java
class Food {}
class Fruit extends Food {}
class Apple extends Fruit {}
class Orange extends Fruit {}
class RedApple extends Apple {}
~~~



## `List<Apple>` IS-NOT-A `List<Fruit>`

面对`Apple`和`Fruit`而言，`Apple` IS-A `Fruit`是个真命题，但是对于它们的容器而言，`List<Apple>` IS-A `List<Fruit>`却是个假命题，因为`List<Apple>`类并没有从 `List<Fruit>`继承到任何的方法或者field。

正因如此，下面的两行代码一个编译成功一个编译失败

~~~java
Fruit maybeApple = new Apple();

List<Fruit> maybeApples = new ArrayList<Apple>();  // 编译失败
~~~



## 上界通配符

想要让对象的引用达到 **<u>编译期只确定是个Fruit的容器，具体是什么Fruit的容器留到运行期决定</u>** 这样的效果，需要使用上界通配符

~~~java
List<? extends Fruit> kindOfFruit = new ArrayList<Apple>();
~~~

对于上界通配符而言，有一些限制，例如：

- `kindOfFruit`除了null，无法add任何类型的对象，因为编译器在在编译期无法通过`List<? extends Fruit>`声明来确定`?`具体是什么类型，`?`可以是`Fruit`，也可以是`Apple`，还可以是`Orange`等等子类型，所以编译器索性不允许你add任何类型

  > 在赋值语句中，等号左侧的内容能在编译器确定，等号右侧的内容只能在运行期确定

- `kindOfFruit`可以调用get方法，但返回值只能赋值给Fruit类型或者Fruit父类型的引用



## 下界通配符

同样的，想要让对象的引用达到 **<u>编译期只确定是个Apple父类的容器，具体是哪个父类留到运行期决定</u>** 这样的效果，需要使用下界通配符

~~~java
List<? super Apple> maybeApples = List<Fruit>();
~~~

下界通配符也有限制：

- `maybeApples`可以调用add方法，但必须是`Apple`或者`Apple`的<u>**子类型**</u>，因为`maybeApples`只能存放`Apple`的父类型，具体是什么父类型运行期确定，所以编译器只允许你存放满足 is-a `Apple`的类型
- `kindOfFruit`可以调用get方法，但返回值只能赋值给`Object`，因为`?`的范围只有下界，没有上界



## 等号右侧的内容只能在运行期确定

<u>**等号右侧的内容只能在运行期确定**</u> 这句话可能会令人困惑，可以参考下面这种情况——

~~~java
Fruit fruit = new Random().nextBoolean() ? new Apple() : new Orange();
~~~

这段代码显然是合法的，能通过编译的，但是`fruit.getClass()`只能等运行了以后才知道，编译器无法分析表达式，所以只能把任何等号右侧的表达式结果当作未知来看到