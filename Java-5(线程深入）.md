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

TODO



#### 锁的类型

1. 无锁状态
2. 偏向锁 拿到锁的线程大概率是第一个拿到锁的线程，只判断锁是否当前拿到线程（被撤销需要其他线程的来修改）
3. 轻量级锁 CAS
4. 重量级锁



优化：

以上4点逐步升级，还有**锁消除、锁粗化、逃逸分析**



#### DCL

双重检查锁定