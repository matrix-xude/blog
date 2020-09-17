---
title: Java线程
categories:
  - Java
tags:
  - Java
date: 2020-09-14 18:20:08
---



#### Thread基础

+ 启动方式只有2种（Thread , Runnable）,Thread 源码上说明的，其它方式都来源这2种



#### Thread状态

+ 线程的状态有6种

![线程状态图](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqZ0gAkbUdZWXTjpW4ZzBlYlUv94lyPG15P*93YPlOJH7K01lnaXTSHLp73AuZAQlhLRQ5jHK2Vsob0Hho3Go.Yg!/b&bo=WAacAwAAAAABB.E!&rf=viewer_4)

+ 阻塞状态仅有 synchronized 能实现，Lock底层调用的是 LockSupport.unpark(Thread) 、 LockSupport.parkUntill()



##### 死锁

1. 条件互斥
2. 保持请求
3. 不剥夺
4. 环路请求

##### 活锁

线程1 拿到资源A，等待资源B

线程2 拿到资源B，等待资源A

然后2个线程不断拿锁释放锁，直到某个线程拿到2把锁



#### CAS原理（Compare And Swap）

Atomic 原子性：不可再分，如synchronized，内部所有语句不可再分，要么都执行，要么不执行

利用现代CPU的 cas 指令，不断的循环比较旧值是否等于拿到的旧值，不同就重新执行

##### 悲观锁

一开始就拿到锁，执行完毕再释放锁

##### 乐观锁

先执行完毕，再用cas指令进行比较，如果比较结果不同，再次执行，直到比较初始条件相同

1. ABA问题，初始值比较
2. 开销问题
3. 只能保证一个共享变量的原子性



**悲观锁效率小于乐观锁，CAS指令**（线程挂起的时间 3-5ms, cas指令 0.6 ns）

AtomicInteger,AtomicBoolean 原子级别的应用

AtomicMarkableReference 是否改变,AtomicStampedReference改变过几次



#### BlockingQueue

阻塞队列，并非所有方法都阻塞

1.  add(E)、remove(E) 是一对，满或者没有的时候抛出异常
2.  offer(E)、poll()是一对，返回false 或者 null
3.  **put(E)、take()**才会真正阻塞

有界队列：规定了最大容量

无界队列：没有规定最大容量



主要的实现类：

1. ArrayBlockQueue 一个由数组结构组成的有界阻塞队列
2. LinkedBlockingQueue 一个由链表结构组成的有界阻塞队列
3. PriorityBlockingQueue 一个支持优先级排序的无界阻塞队列
4. DelayQueue 一个根据延时排序的无界阻塞队列，如果延迟时长没到，消耗者取不到容器内的数据
5. SynchronousQueue 一个不存储元素的阻塞队列，放入之后必须要一个消耗者拿走
6. LinkedTransferQueue 一个由链表结构组成的的无界阻塞队列，主要是多了 transfer方法，放入之前查看是否有消耗着等待，如果有，直接传给消耗者，没有则阻塞线程，等待消耗者。tryTransfer：尝试给消耗者，没有则放入队列
7. LinkedBlockingDeque 一个由链表结构组成的双向无界队列，两边可以放入、取出



#### 线程池

![](http://m.qpic.cn/psc?/V13IATxj2uFujC/bqQfVz5yrrGYSXMvKr.cqRJzUSNRaWo0Eoj2y14yQDIqOjh6uVXqrevVjKAaUwwBcln.iBSdYJ9bJB2Q*vZ37fvscBLLR64VGp1UVSXmpTo!/b&bo=ewQ4BAAAAAABB2M!&rf=viewer_4)

1. 优先使用核心线程数 corePool ,执行线程
2. 如果核心线程不足，放入BlockingQueue(Runnable)中等待执行，大部分系统给的队列都是 Integer.MAX_VALUE
3. BlockingQueue队列满了之后，才开启大于 corePool,小于maximumPool的其它线程开始执行
4. 如果再次溢出，则需要用到拒绝策略，RejectedExecutionHandler



##### 执行线程

1. execute 无返回值
2. submit 返回Feture<V>

##### 关闭线程池

1. shutdown 仅仅关闭线程池，不中断正在执行的线程，也不移除BlockingQueue队列
2. shutdownNow 关闭线程池，移除BlockingQueue队列中的数据，同时中断正在执行中的线程，不一定成功，仅仅发出一个 interrupt 信号，自己处理中断

##### 创建线程池的特性

1. CPU密集型 

   线程数 ：机器的核心数 * 1

   一般来说，最大线程数为`Runtime.getRuntime().availableProcessors()`  + 1

   为什么是 +1 ，因为可能出现”页缺失“状态（读取磁盘虚拟内存导致当前线程不做事挂起，等读取到磁盘数据），保证CPU核心数

   全部利用起来

2. IO密集型

   线程数 ：机器的核心数 * 2