---
title: "Java基础知识3-多线程相关"
date: 2020-10-07T14:23:52+08:00
tags: [java基础]
draft: true
---

---
<!--more-->
# JUC包

- locks
- atomic
- sync
- collections
- executors

# 线程的状态

- 新建（NEW）
  
  表示线程被创建出来还没真正启动的状态，可以认为它是个 Java 内部状态。
  
- 就绪（RUNNABLE）
  
  表示该线程已经在 JVM 中执行，当然由于执行需要计算资源，它可能是正在运行，也可能还在等待系统分配给它 CPU 片段，在就绪队列里面排队。
  
- 阻塞（BLOCKED）
  
  获取锁的过程。
  
- 等待（WAITING）
  
  等待其它线程采取某些操作，例如消费者模式。

- 计时等待（TIMED_WAIT）
  
  有超时时间的等待状态。
  
- 终止（TERMINATED）
  
  线程完成工作，结束状态。

## 1.0 java.util.concurrent.locks
> 对 synchronizd、wait、notify等进行补充、增强。

### 1.0.1 Interfaces
- Condition
- Lock
- ReadWriteLock

#### Condition
> 可以看成对 Object `wait()、notify()、notifyAll()` 方法的扩展替代。

#### Lock
> 可以视为 `synchronized` 的增强版，提供了更灵活的功能。提供了现实锁等待、锁中断、锁尝试等功能。

`lock()` 和 `lockInterruptibly()`

lock() 方法类似于使用 `synchronized` 关键字加锁，如果锁不可用，出于线程调度目的，将禁用当前线程，并且在获得锁之前，将该线程一直处于休眠状态。

lockInterruptibly() 方法是如果锁不可用，那么当前正在等待的线程是可以被中断的，比 `synchronized` 关键字更灵活。

#### ReadWriteLock
> 

### Classes
- AbstractOwnableSynchronizer
- AbstractQueuedLongSynchronizer
- AbstractQueuedSynchronizer
- LockSupport
- ReentrantLock
- ReentrantReadWriteLock
- StampedLock

