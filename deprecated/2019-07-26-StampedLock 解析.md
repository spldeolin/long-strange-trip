---

title: StampedLock 解析

date: 2019-07-26 08:54

updated: 2019-07-26 08:54

tags:
- 并发

categories: Java

permalink: stampedlock

---

过时原因：教学类文章

## 简介

`StampedLock`是1.8新提供的读写锁

## 比原来的ReentrantReadWriteLock好在哪里？

1. ReentrantReadWriteLock的读锁是悲观锁，在多读少写的情况下，写线程不容易抢到锁；StampedLock通过乐观读锁缓解了这个问题
2. Stamped有一个锁升级机制，能提高if-then-update场景的并发量

## 线程等待锁是怎么实现的？

基本上都是CAS，不行就自旋，再不行就进CLH队列然后park线程

## CLH队列

`StampedLock`内部持有了一个CLH队列，所有申请锁失败的线程都会记录在队列中，一个节点对应一个线程，节点内部持有一个标志位，用来判断当前线程是否释放了锁。

当一个线程试图获取锁的时候，它会以CLH的队尾节点作为自己的前序，然后向前遍历所有前序节点，看他们是否已经成功释放锁了

## writeLock

- 独占，不可重入
- 没有任何线程持有读锁或者写锁时，才能获取锁，否则等待
    - 内部采用CAS修改`STATE`状态，失败就进`acquireRead`方法自旋尝试，再失败就加入队列，park线程
- 获取成功将会返回`stamp`
- 一旦获取，其它的线程请求读锁或是写锁时必须等待

## readLock

- 共享，不可重入
- 没有任何线程持有写锁时，才能获取锁，否则等待
- 获取成功将会返回stamp
- 一旦获取，其它线程请求写锁时必须等待

## 乐观读的流程

`StampedLock`的乐观读锁适用于**读取资源**的场景。

```java
// 这段代码是StampedLock类注释上的demo
double distanceFromOrigin() {

        long stamp = sl.tryOptimisticRead(); // no.1 
        
        double currentX = x, currentY = y; // no.2
        
        if (!sl.validate(stamp)) { //no.3
            // 如果被抢占则获取一个共享读锁（悲观获取）（4）
            stamp = sl.readLock();
            try {
                // 将全部变量拷贝到方法体栈内（5）
                currentX = x;
                currentY = y;
            } finally {
                // 释放共享读锁（6）
                sl.unlockRead(stamp);
            }
        }
        // 返回计算结果（7）
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }
```

1. 由于乐观读锁，所以要先在`no.1`处获取版本号，
2. `no.2`读取资源，虽然读取x和y是在一行代码里，但他们依然是非原子操作。不过这里乐观地认为期间资源没有被修改
3. `no.3`用于校验在`no.1`至`no.3`期间，有没有线程获取过写锁，如果没有，乐观获取的值成立。
    - 如果有其它线程获取写锁后释放，会导致validate方法return false
    - return false的原因是validate方法会比较获取读锁时的stamp与当前的lock.stamp是否一致，而其它线程释放写锁的时会更新lock.stamp的值
    - 线程释放写锁的会更新lock.stamp是因为写锁的设计意图是修改资源时获取的，所以悲观地认为释放写锁时，资源是被用户代码修改了。

4. 如果`no.3`失败，这里没有设计成自旋，而是直接获取悲观读锁，以互斥写锁的方式来读取资源的值

## 悲观读的流程

`StampedLock`的乐观读锁配合锁升级适用于**if-then-update**的场景。

```java
  void moveIfAtOrigin(double newX, double newY) { 
     
     long stamp = sl.readLock(); // no.1
     try {
       while (x == 0.0 && y == 0.0) { // no.2
         long ws = sl.tryConvertToWriteLock(stamp); // no.3
         if (ws != 0L) {
           stamp = ws;
           x = newX;
           y = newY;
           break;
         }
         else {
           sl.unlockRead(stamp);
           stamp = sl.writeLock();
         }
       }
     } finally {
       sl.unlock(stamp);
     }
   }
 }}
```

1. 用户业务代码是`if (x==0.0&&y==0.0) {x=newX; y=newY;}`，典型if-then-update的业务，示例中的其它代码都是在操作锁
2. 由于是先读再写，所以没有非常粗暴地直接上写锁，而是先上读锁，提高了no.2的if表达式的并发量，等到了修改资源的临界区，才让所有的读线程去争夺升级成写锁。
3. 升级写锁失败的读线程会直接去获取写锁，然后重试try-then-update
4. 为什么不用乐观读？

    乐观失败的情况，还是需要升级成悲观读或是自旋，代码会比较复杂

5. StampedLock处理if-then-update的思路是尽可能缩小写锁的临界区，提高整个代码的并发量。