---
title: For-Each循环

date: 2017-04-03 07:43:00

updated: 2017-04-03 07:43:00

tags:

categories: Java

permalink: for-each-loop
---

For-Each是Java中For-Index的一种加强，是Java 5带来的新语法糖。

#### 什么场合应该使用它

For-Each似乎并不是必须的，但很多场合下使用它十分合适。
在实际开发中，经常会出现需要遍历数组，或是Collection容器的情况，就像source1那样。

```java
/**
 * source1
 */
String[] args = {"a", "b", "c"};
for (int i = 0; i < args.length; i++) {
	String arg = args[i];
	System.out.println(arg);
}
```

source1中的for循环仅仅希望得到args数组中的每一个元素，但是看看为此做了多少额外的事情。

第一件事。在循环了声明了下标i。真正归 For-Index 负责的任务其实是在满足要求的情况下不停地i++，所以在{}代码块中定义的代码块更像是一种附带——由于i在不断自增，那么在代码块中“顺便”可以取到args中的每一个值。

第二件事。For-Index 只关注int i。所以想要取得每一个元素，还得声明一个arg变量来引用每一个元素。（虽然source1中，arg只被用到了一次，但实际开发中常常不只如此，根据DONT REPEAT YOURSELF原则，arg是必要的）

而使用For-Each，在编码上解决了这两个问题。

```java
/**
 * source2
 */
String[] args = {"a", "b"};
for (String arg : args) {
	System.out.println(arg);
}
```

For-Each直接()中声明了arg引用，不需要在代码块中专门声明。int i也不再必要了，For-Each会循环到args中无值可取为止。

显然，单纯为了遍历数组或容器对象中的每个元素，For-Each比For-Index在编码上更合适。在可读性方法，For-Each很容易让人知道设计者希望遍历冒号后面对象的全部元素。

> Deolin在一开始并不习惯使用For-Each，直到忍受不了符合格式规约的For-Index需要在()中写大量的空格。

#### 哪些类型的对象可以适用For-Each

- 数组
- Collection类
- 任何实现了Iterable接口的自定义类

（根据面向接口的思想，Deolin习惯把第三类对象称之为“可迭代的”对象）
第一类，第二类在实际开发中经常用到，而第三类能够适用For-Each的原因需要通过源码来进行分析。

#### For-Each的原理是什么

```java
/**
 * source3
 */
String oneArg = "a";
for (String arg : oneArg) {}
```

发生了一个编译错误——Can only iterate over an array or an instance of java.lang.Iterable，For-Each无法遍历数组和“可遍历的”实例以外的对象。
那么调查一下java.lang.Iterable的source，这个接口只有一个方法声明——Iterator<T> iterator()（Java 8之后又追加了两个default方法），也就是说，适用For-Each的对象实际上都实现了Iterator<T> iterator()方法。
For-Each是通过调用Iterator obj的iterator()方法获得一个“迭代子”（Iterator对象）来实现迭代的。
即，身为了一个“可迭代的”对象，必须实现“返回一个迭代子”的方法，让外部能借此来迭代自己。

For-Each取得Iterator对象之后，会反复调用hasNext()和next()方法，原理与while类似

```java
/**
 * source4
 */
List<String> list = new ArrayList<String>();
list.add("a");
list.add("b");
list.add("c");

Iterator iterator = list.iterator();
while (iterator.hasNext()) {
    String s = (String) iterator.next();
    System.out.println(s);
}
```

在java.util.ArrayList$Itr的hasNext()和next()中添加两个断点可以印证这点。

#### 利用For-Each的原理定制化迭代策略

现在定义一个List（source5），如果想要实现倒序输出，通过代理模式定制一个反迭代器似乎是个不错的方法。
```java
/**
 * source5
 */
List<String> list = new ArrayList<String>();
list.add("a");
list.add("b");
list.add("c");
list.add("d");
```

首先，定义一个代理类，使其能够持有一个Collection对象，并代理它的行为。
并且在定义一个向For-Each提供Iterable对象的对外方法。

    /**
     * source6
     */
    class ReversibleList<T> {
        // 持有一个List
        private List<T> list;
    
        public ReversibleList(List<T> list) {
            this.list = list;
        }
    
        public int size() {
            return list.size();
        }
    
        public T get(int i) {
            return list.get(i);
        }
    
        public Iterable<T> getIterableObj() {
            return new Iterable<T>() {
    
                @Override
                public Iterator<T> iterator() {
                    // 定制的迭代器
                    return new Iterator<T>() {
    
                        private int index = size() - 1;
    
                        @Override
                        public boolean hasNext() {
                            return index > -1;
                        }
    
                        @Override
                        public T next() {
                            return get(index --);
                        }
    
                    };
                }
    
            };
        }

测试一下

```java
    /**
     * source7
     */
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        list.add("a");
        list.add("b");
        list.add("c");
        list.add("d");
        ReversibleList<String> rl = new ReversibleList<String>(list);
        for(String s : rl.getIterableObj()) {
            System.out.println(s);
        }
    }
```

输出结果

    d
    c
    b
    a
