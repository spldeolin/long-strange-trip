---
title: 多线程

date: 2017-04-04 10:06

updated: 2017-04-04 10:06

tags:

categories: Java

permalink: thread
---

> “如果有机会成为一个系统级的程序员，线程的机制必须了解的非常透彻才可以。”

###### 1、线程是程序里不同的执行路径，进程的执行 指的是进程中的主线程开始执行了

###### 2、进程是一段可以独立运行的程序，线程是进程的一个实体

###### 3、Java中线程代表了一个对象，实现了Runable接口，或是继承了Thread类的类对象，都可以称之为“线程”，原因是这两种类对象内部都有run()方法。

run()中的代码块，表示这个线程的执行路径，线程一旦开始执行，run()中的代码块便会与其他正在执行的线程并发，而在实际开发中需要被重写（或者说实现）的，便是这个方法。

```java
/**
 * source1
 */
class Producer extends Thread {

    @Override
    public void run() {
        for (int i = 0; ; i++) {
            System.out.println("produced No." + i);
        }
    }

}
```

```java
/**
 * source2
 */
class Consumer implements Runnable {

    @Override
    public void run() {
        for (int i = 0; ; i++) {
            System.out.println("destroyed No." + i);
        }
    }

}
```

######  4、让一个线程开始执行的做法是调用它的start()方法。（如果他有的话）

对于继承自Thread类的线程类，start()存在于他的父类Thread中，可以直接调用。
对于实现了Runable接口的线程类，没有start()可调用，合适的做法是将对象注入到一个Thread类对象中，然后调用代理的start()方法。

```java
/**
 * source3
 */
void startThreads() {
    Producer producer = new Producer();
    producer.start();

    Consumer consumer = new Consumer();
    new Thread(consumer).start();
}
```

######  5、让一个线程开始执行的方法是start()，而不是run()，后者仅仅是调用方法，前者才是启动一个线程。

![](/static/img/6a58a4067da9561d42290452088306ab.png)

###### 6、start()方法本质上是让线程进入就绪状态，就绪状态代表着等待接受CPU的调度。等到CPU将时间片分给该线程的时候，线程才真正进入运行状态。由于调度发生在CPU内部且无法受到Java代码的控制，所以可以认为start()调用后，线程就开始运行了。

 run()方法一旦返回，代表线程终止

###### 7、sleep()方法可以在线程外部和run()的内部调用，常常用来让线程等待若干秒再执行后续的代码。

    /**
     * source4
     */
    void startWithSleepping() {
        Producer producer = new Producer();
        producer.start();
        try {
            // producer进入睡眠1s
            producer.sleep(1000);
        // 任何打断睡眠的情况都会捕获到这个异常
        } catch (InterruptedException e) {
            System.out.println("Something breaks sleepping");
        }
    }

```java
/**
 * source5
 */
class Producer extends Thread {

    @Override
    public void run() {
        for (int i = 0; ; i++) {
            System.out.println("produced No." + i);
            try {
                // 让自身进入睡眠1s
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                System.out.println("Something breaks sleepping");
            }
        }
    }

}
```

######  8、join()方法作用是让该线程认为“外部调用自己的run()方法而不是start()”，外部会陷入阻塞状态，直到run（）方法返回。

###### 9、yield()方法作用是让被调用的线程主动释放时间片，进入就绪状态，等待CPU的调度。sleep()方法也能起到类似的效果。

###### 10、四类常用线程池。

- newSingleThreadExecutor()
- newFixedThreadPool()
- newCachedThreadPool()
- newScheduledThreadPool()

```java
/**
 * source6
 */
void startByPool() {
    ExecutorService pool = Executors.newCachedThreadPool();
    // 注入线程类并执行
    pool.execute(new Producer());
    pool.execute(new Consumer());
    // 关闭线程池
    pool.shutdown();
}
```