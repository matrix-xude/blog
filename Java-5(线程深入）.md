---
title: Java线程深入
categories:
  - Java
tags:
  - Java
date: 2020-09-14 18:20:08
---



#### 什么是AQS

AbstractQueuedSynchronizer，锁的基本实现原理，很重要

AQS用的是**模板方法**设计模式

如何自己实现一个同步锁：

继承AbstractQueuedSynchronizer，覆写tryAcquire, tryRelease

compareAndSetState()，原子操作比较，设置独占锁



#### AQS的基本思想和CLH队列锁

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqdW3oUFDzjf8kN6jk3nDj9gRME4*9qlXLhpLXi*SHNmjDRPSqnWCkFeWmf0oss.xylA5BCke81HVDd*R7dt234A!/b&bo=jAc4BAAAAAADB5U!&rf=viewer_4)

每个想获取锁的线程打包成节点，自旋N次后挂起，很显然，CLH队列是公平锁

state很关键，重入锁每次+1，释放-1，没拿到锁或者锁的state为0，unlock会报错



#### JMM java内存模型

Java Memory Model

主内存，工作内存（每个线程一份）



#### volatile

1. 只能保证可见性，非原子性操作，因为CPU切换，也有线程安全问题

2. 抑制重排序（CPU流水线，一次行政多个指令）

3. 通过Lock来实现



#### synchronized

1. 使用monitorenter和monitorexit来实现
2. monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处
3. 每个monitorenter必须有对应的monitorexit与之配对 
4. 任何对象都有一个monitor与之关联

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqYypBcO.5GenlEZzCd5mEHgIuYNNA0lDo3MFGVzA4AeUbMmRHcmmc7K0sIaNxYrIL6KXXJF8oOA6ch37Hw7hs0E!/b&bo=9wJ2AAAAAAADB6E!&rf=viewer_4)

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqeUIrS8gOJB.ov3xYx6q2bPMJavYK3s8oxGDo8zngSAdwdmCnsu30WvsaIVkQ.LNTxLcPM5907fb.mh9zdifPOE!/b&bo=RwIBAQAAAAADB2c!&rf=viewer_4)



#### synchronized锁的类型

1. 无锁状态
2. 偏向锁 ：大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。无竞争时不需要进行CAS操作来加锁和解锁。 
3. 轻量级锁：通过CAS操作来加锁和解锁。
4. 重量级锁：阻塞线程

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqSrnB2rISCsaLiJaKgZLOcMqhWopMzpyKLf8s*EYSZigGPanzzU*j0bKN5d0bkfQCiMfM8Pq1bbhF069F3KiEzQ!/b&bo=cAKGAQAAAAADB9c!&rf=viewer_4)



synchronized做了哪些优化？

以上4点逐步升级，还有**锁消除、锁粗化、逃逸分析**



#### DCL

双重检查锁定