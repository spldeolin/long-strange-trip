---
title: 高并发下的单例模式

date: 2019-06-15 10:30

updated: 2019-06-15 10:30

tags:
- Java

categories: Java

permalink: single-in-concurrerncy
---



## 简介

单例模式基本都认识，而这篇POST将会介绍

- 单例模式在并发情况下会遇到哪些问题

- 如何在设计一个单例类，使其在高并发的场景也下具有较好的表现



## 饿汉式

~~~java
class HungrySingleton {

    private static final HungrySingleton instance = new HungrySingleton();

    private HungrySingleton() {
    }

    public static HungrySingleton getInstance() {
        return instance;
    }
}
~~~

优先是类加载的时候就将实例new出来了，不会出现线程安全的问题；

缺点也是类加载的时候就把实例new出来了，如果单例类比较大，又不会立即用到它的实例，就会造成内存浪费



## 懒汉式

~~~java
public class LazySingleton {

    private static LazySingleton instance;

    private LazySingleton() {
    }

    public static LazySingleton getInstance() {
        if (instance == null) {
            instance = new  LazySingleton();
        }
        return instance;
    }
}
~~~

优点是延迟加载，需要用的时候才会创建实例；

致命缺点是`getInstance`方法线程不安全



## 线程安全的懒汉式

~~~java
public class ThreadSafeLazySingleton {

    private static ThreadSafeLazySingleton instance;

    private ThreadSafeLazySingleton() {
    }

    public static synchronized ThreadSafeLazySingleton getInstance() {
        if (instance == null) {
            instance = new ThreadSafeLazySingleton();
        }
        return instance;
    }
}
~~~

通过`synchronized`关键字，确保了线程安全；

但是，用重型锁将整个方法锁住完全没必要，并发较高时性能会下降



## DCL校验

~~~java
class DclSingleton {

    private static DclSingleton instance;

    private DclSingleton() {
    }

    public static synchronized DclSingleton getInstance() {
        if (instance == null) {                          // 第一次校验
            synchronized (DclSingleton.class) {          // 锁
                if (instance == null) {					 // 第二次校验
                    instance = new DclSingleton();
                }
            }
        }
        return instance;
    }
}
~~~

可以看到，把锁的临界区范围缩小到了new实例那行，性能提高了不少；

但是，需要考虑到线程存在暂时获取不到锁的情况。如果线程A在等待锁挂起期间，实例被获得锁的线程new出来了，那么线程A获得锁之后就不应该再次new对象了，所以在临界区需要进行第二次校验



## 指令重排

DCL单例依然存在线程不安全的地方。

`instance = new DclSingleton();`本质上分为了3个步骤

1. 分配空间

2. 初始化对象

3. 指向内存地址

其中，后两个步骤不存在数据依赖关系，也就是说，第2步和第3步可能会应该发生指令重排而调换问题。

考虑一个场景，现在指令重排发生了，线程C在分配空间之后，`instance`先指向了内存地址，`instance != null`成立，对象还未初始化，虽然线程C持有锁，但是其他线程依然可以进行第一次校验，其他线程会return一个未初始化完成的对象，从而引发问题

解决方法很简单，通过volatile阻止指令重排就可以了

~~~java
public class VolatileDclSingleton {

    private static volatile VolatileDclSingleton instance;

    private VolatileDclSingleton() {
    }

    public static synchronized VolatileDclSingleton getInstance() {
        if (instance == null) {
            synchronized (VolatileDclSingleton.class) {
                if (instance == null) {
                    instance = new VolatileDclSingleton();
                }
            }
        }
        return instance;
    }
}
~~~

懒汉式写到这个地步，算是滴水不漏了。



## 静态内部类

~~~java
public class InnerSingleton {

    private static class Inner {
        public static final InnerSingleton instance = new InnerSingleton();
    }

    private InnerSingleton() {
    }

    public static InnerSingleton getInstance() {
        return Inner.instance;
    }
}
~~~

跟饱汉式很像，区别在于实例字段被声明在静态内部类里了，只有内部类被类加载的时候才会new实例

而内部类的类加载时机被封装进了`getInstance`方法，达到了“懒加载”的效果

**这是一种非常棒、非常值得推荐的一种单例实现方式**



## 枚举

~~~java
public class EnumSingleton {

    private EnumSingleton() {
    }
	
    private enum Inner {
        INSTANCE_ENUM;
        
        private EnumSingleton instance;
        
        Inner() {
            instance = new EnumSingleton();
        }
        
        public EnumSingleton instance() {
            return instance;
        }
    }
    
    public static EnumSingleton getInstance() {
        return Inner.INSTANCE_ENUM.instance();
    }
}
~~~

按照《Effective Java》的说法，枚举实现是**最好**的实现方式

1. 首先，内部枚举类的类加载时机也被封装进了`getInstance`方法，他是懒加载的

2. enum里的构造方法默认是私有的，`Inner(){}`等同于`private Inner(){}`

3. enum里的字段默认是`static final`的，所以`INSTANCE_ENUM`只会被初始化一次

4. 前几种方式序列化的时候会去序列化那个instance字段，这可能不是想要的效果，但是枚举则不会被序列化处理到

5. 类加载时new实例，所以线程安全，且无锁

6. 唯一的缺点可能是涉及到枚举，会比前几种方式难以理解一点....

   