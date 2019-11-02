---
title: JUC并发框架中ThreadPoolExecutor的关键配置项

date: 2018-10-07 12:56

updated: 2018-10-07 12:56

tags:
- 并发

categories: Java

permalink: thread-pool-properties
---



## core pool size

代表线程池的大小，ThreadPoolExecutor对象每execute()/submit()一个任务，都会创建一个新的线程，直到线程数填满core pool size。



core pool size被填满后，新的任务会被放置到阻塞队列。



## maximum pool size

代表可活动的线程的上限，ThreadPoolExecutor中的线程数无论如何都不会超过这个值。



## keep alive time

代表空闲线程的存活时间，当池中一个线程没有任务执行的时间超过了keep alive time，将被标记为**回收候选者**，当线程数大于core pool size时，所有的**回收候选者**线程将被终止。



## blocking queue

代表阻塞队列，线程值线程数达到core pool size后，新任务会被放置到队列中。如果队列被任务填满了，新任务会移交给线程池，线程池继续创建新线程，直到线程池大小抵达maximum pool size，而如果还有更多的任务，则会被采取**饱和策略**



blocking queue有3种

1. 无限队列：可放置无穷多的任务，高并发时会占用大量内存，有OOM的风险
2. 有限队列：队列被任务填满后，新任务会直接移交给线程池
3. 同步移交：认为不是“阻塞队列”，任务加入队列后直接移交给线程池，如果池中线程数等于maximum，采取**饱和策略**



## 线程数的动态调整

ThreadPoolExecutor源码的Java Doc中总结性地描述了线程池对线程数的动态调整，Deolin将这些描述翻译在下面

~~~
ThreadPoolExecutor会根据maximum pool size和core pool size自动地调整线程数。
当线程数小于core pool size时，即使有空闲线程，也为新提交的任务创建新线程；
当线程数大于core pool size，小于maximum pool size时，仅仅在阻塞队列满的情况下才创建新线程；	
当线程数大于core pool size时，空闲线程将会被回收。
~~~



## 演示

~~~
@Log4j2
class TimeConsumingRunnableTask implements Runnable {

    @Override
    public void run() {
        log.info("线程 {} 开始执行", Thread.currentThread().getName());
        try {
            Thread.sleep(10_000);
        } catch (InterruptedException e) {
            log.error(e);
        }
        log.info("线程 {} 执行结束", Thread.currentThread().getName());
    }

}
~~~



~~~java
    int coreSize = 2;
    int maxSize = 5;
    long timeout = 10L;
    int queueSize = 10;
    ThreadPoolExecutor executor = new ThreadPoolExecutor(coreSize, maxSize, timeout, TimeUnit.SECONDS, new ArrayBlockingQueue<>(queueSize));

    // 填满coreSize （2）
    FutureTask futureTask = new FutureTask<>(new TimeConsumingCallableTask());
    executor.execute(futureTask);
    executor.execute(new FutureTask<>(new TimeConsumingCallableTask()));

    // 填满阻塞队列
    for (int i = 0; i < queueSize; i ++) {
        executor.execute(new FutureTask<>(new TimeConsumingCallableTask()));
    }

    // 填满maxSize （5 - 2）
    executor.execute(new FutureTask<>(new TimeConsumingCallableTask()));
    executor.execute(new FutureTask<>(new TimeConsumingCallableTask()));
    executor.execute(new FutureTask<>(new TimeConsumingCallableTask()));

    // 被拒绝（调用者执行策略）
    executor.setRejectedExecutionHandler(new CallerRunsPolicy());
    executor.execute(new FutureTask<>(new TimeConsumingCallableTask()));

    try {
       futureTask.get();
    } catch (InterruptedException e) {
        log.error(e);
    } catch (ExecutionException e) {
        log.error(e.getCause());
    }

    log.info("SUCCESS");
~~~



