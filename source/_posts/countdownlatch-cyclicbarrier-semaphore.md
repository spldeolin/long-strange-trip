---

title: CountDownLatch、CyclicBarrier、Semaphore详解

date: 2019-10-11 17:42

updated: 2019-10-11 17:42

tags:
- Java

categories: Java

permalink: countdownlatch-cyclicbarrier-semaphore

---

## 简介

介绍一下并发包里比较常用的3个工具——`CountDownLatch`、`CyclicBarrier`、`Semaphore`



## CountDownLatch

`CountDownLatch`可以理解为倒计时器，他具有以下特性

- 每个`CountDownLatch`对象都持有了一个大于0的int值作为“倒计时”，在构造方法中被初始化
- 调用`CountDownLatch`对象的`countDown`方法，可以使倒计时自减1
- 倒计时最小只会被减到0
- 调用`CountDownLatch`对象的`await`方法，当前线程会被阻塞直到倒计时被其它线程减到0。
- 当倒计时已经为0时，调用`await`方法不会被阻塞



`CountDownLatch`是通过AQS实现的，Shared结尾的两个try方法被重写了，所以本质就是个共享锁，不复杂

## CyclicBarrier

`CyclicBarrier`可以想象成为日剧中常常出现的「鹿威」

![](/images/countdownlatch-cyclicbarrier-semaphore-01.gif)

它具有以下特性

- 每个`CyclicBarrier`对象都持有一个int值作为“容量”，在构造方法中被初始化
- 调用`CyclicBarrier`对象的`await`方法，当前线程会被阻塞，同时`CyclicBarrier`的当前容量会自增1（想象成一滴水流进了「鹿威」）
- 当`CyclicBarrier`对象被阻塞的线程数达到容量时，被阻塞线程都会被唤醒（想象成「鹿威」内的水足够多了，水都被放了出去，源码里将这个现象命名为`tripped`（倾翻））
- `CyclicBarrier`每轮唤醒线程后，下一轮再次调用`await`又会阻塞线程



`CyclicBarrier`的核心代码在`dowait`方法内，底层是通过`ReentrantLock`结合`Condition`来实现阻塞与倾翻线程的。

具体实现是每lock一个线程，就把内部的剩余容量属性自减1，当减到0时就用`Condition`对象唤醒全部线程

## Semaphore

`Semaphore`是共享版的`ReentrantLock`，

跟`ReentrantLock`的区别在于底层的AQS重写了两个`try***Share`方法而不是`try***`方法。