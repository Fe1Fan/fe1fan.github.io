---
title: "JVM 笔记2：垃圾回收"
date: 2020-10-07T14:23:52+08:00
tags: [jvm, java]
---

# JVM 垃圾回收

- 什么是垃圾
- 哪里需要垃圾回收
- 什么时候回收
- 怎么回收

## 什么是垃圾
当对象不再被需要的时候，就是需要被回收的垃圾。

## 哪里需要垃圾回收
1. Java堆

2. 方法区


## 什么时候回收
1.年轻代

当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC，也就是新生代的GC。

2.老年代

在发生Minor GC之前，虚拟机必须先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那这一次Minor GC可以确保是安全的。如果不成立，则虚拟机会先查看-XX：HandlePromotionFailure参数的设置值是否允许担保失败（Handle Promotion Failure）；如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者-XX：HandlePromotionFailure设置不允许冒险，那这时就要改为进行一次Full GC。

3. 方法区
方法区主要回收两部分
- 废弃的常量
- 不再被使用的类型

常量：

当方法区的常量不再被任何一个对象引用时

不再被使用的类型：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。

- 加载该类的类加载器已经被回收。

- 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。



## 垃圾回收的步骤
1.收集

2.清理

## 1.引用计数法（Reference Counting）

JVM 并没有采用大多数人所说的引用计数法，是从来没有。

### 概念
在对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加一；当引用失效时，计数器值就减一；任何时刻计数器为零的对象就是不可能再被使用的。

### 优点
原理简单，判定效率也很高

### 缺陷
占用了一些额外的内存空间来进行计数

```java
class Demo {
    public Object object = null;
    
    public static void main(String[] args){
      Demo demoA = new Demo();
      Demo demoB = new Demo();
      demoA.object = demoB;
      demoB.object = demoA;
      
      demoA = null;
      demoB = null;
      //此时 demoA 和 demoB 都不会被回收
      System.gc();
    }
}
```