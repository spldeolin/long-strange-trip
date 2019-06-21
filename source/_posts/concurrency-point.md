---

title: Java并发 知识点

date: 2019-06-21 15:32

updated: 2019-06-21 15:42

tags:
- Java知识点

categories: Java

permalink: concurrency-point

---

## 基本

1. 线程 and 进程？
2. 线程有哪些状态？
3. `synchronized`是怎么实现的？
4. `synchronized`锁升级优化？
5. `AQS`



## volatile关键字

1. `volatile`是什么？三个特点？
2. `JMM`关于同步的规定？
3. `volatile`可见性是什么？
4. 原子性是什么？
5. `volatile`为什么不保证原子性？
6. 有什么办法保证原子性？
7. 指令重排是什么？
8. `volatile`禁止指令重排的原理？
9. 哪里能用到`volatile`？



## CAS

1. `CAS`是什么？
2. 底层原理？
3. `CAS`有哪些缺点？
4. 如何解决`ABA`问题



## 容器类

1. `ArrayList`为什么线程不安全？
2. 线程安全的`List`？
3. `CopyOnWriteArraySet`底层实现？
4. `ConcurrentHashMap`



## 锁

1. 公平锁 and 非公平锁？
2. 可重入锁（递归锁）
3. 自旋锁
4. 自旋锁的缺点
5. 手写自旋锁
6. 独占锁
7. 读写锁
8. `Lock`和`synchronized`的区别



## 阻塞队列

1. 阻塞队列是什么？
2. 见过哪些阻塞队列？

3. 手写无锁生产者消费者模型



## 线程池

1. 为什么用线程池，优势是什么？
2. 常见线程池
3. 7大参数
4. 拒绝策略
5. 用过Executors创建过线程池吗？
6. 如何配置线程池？



## 死锁

1. 手写一个死锁
2. 怎么排查死锁？



## JUC工具

1. `CountDownLatch`用过吗？
2. `CyclicBarrier`用过吗？
3. `Semaphore`用过吗？
4. `LongAdder`
5. `StampedLock`



## 发音

synchronized / synchronous

terminated

exit

abstract

queue

volatile

field

atomic

concurrent

reentrant

interrupt

cache

maximum

available

stack