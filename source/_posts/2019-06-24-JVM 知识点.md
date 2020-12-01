---

title: JVM 知识点

date: 2019-06-24 16:23

updated: 2019-06-24 16:23

tags:
- 知识点

categories: Java

permalink: jvm-point

---

## JVM内存模型

1. JVM and JMM？
2. 栈、本地方法栈？
3. 程序技术器
4. 堆
5. 方法区（Java8）
6. 线程独占？



## 类加载

1. 是什么？终产物？
2. 步骤
3. 双亲委派模式
4. 为什么用双亲委派模式



## 垃圾回收算法

1. 引用计数
2. 复制
3. 标记清除
4. 标记整理（标记压缩）
5. 为什么要分代



## 如何确定是否是垃圾

1. GC Root是什么
2. 可达性分析



## 垃圾回收器

1. UseSerialGC
2. UseParNewGC
3. UseParallelGC
4. UseConcMarkSweepGC
5. UseParallelOldGC
6. UseG1GC
7. Server / Client模式区别
8. 选什么垃圾回收器？



## G1GC

1. 特点
2. 底层原理
3. 收集步骤



## JVM调优

1. `jps`、`jinfo`
2. `-XX:+PrintGCDetails`、`PrintCommandLineFlags`
3. `-XX:+PrintFlagsInitial`、`-XX:+PrintFlagsFinal`
4. `-Xms`、`-Xmx`、`-Xss`、`-Xmn`是单X参数吗
5. 其它分区大小类参数



## 其它

1. 强、软、弱、虚引用
2. 见过哪些OOM种类？



## 发音

metaspace

null

minor

survivor

serial

initial

stack

tenuring

threshold

exceeded

region

parallel

