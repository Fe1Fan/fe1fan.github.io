---
title: "Java基础知识1-引用和传递"
date: 2020-10-07T14:23:52+08:00
tags: [java基础]
---

---
<!--more-->

## 1.0 引用类型
- 强引用
- 弱引用
- 软引用
- 幻象引用

### 1.0.2 不同引用区别

> 对象不同的可达性（reachable）状态和对垃圾收集的影响。

### 1.0.3 强引用

```java
Object o = new Object;
```

通过`new`关键字创建的对象引用。GC回收垃圾时会决定对象是否可以被回收的引用。

### 1.0.4 弱引用

```java
Object o1 = new Object();
WeakReference<Object> objWeakReference = new WeakReference<>(o1);
Object o2 = objWeakReference.get();
```

通过`WeakReference`实现，在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

可以用内存敏感的缓存。

### 1.0.5 软引用

```java
Object o1 = new Object();
SoftReference<Object> objSoftReference = new SoftReference<Object>(o1);
Object o2 = objSoftReference.get();
```

通过`SoftReference`实现，只有当 JVM 认为内存不足时，才会去试图回收软引用指向的对象：即JVM 会确保在抛出 OutOfMemoryError 之前，清理软引用指向的对象。

软引用通常用来实现内存敏感的缓存。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。

### 1.0.6 幻象引用


```java
Object o1 = new Object();
ReferenceQueue queue = new ReferenceQueue ();
PhantomReference pr = new PhantomReference (o1, queue); 
```

通过`PhantomReference` 实现，无法通过虚引用访问对象的任何属性或函数。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

可用来跟踪对象被垃圾回收器回收的活动，当一个虚引用关联的对象被垃圾收集器回收之前会收到一条系统通知。

## 1.1 值传递

> 除了基本类型之外的，所有类型，包括数组的赋值，都是引用传递

基本类型和引用类型:

```java
// 基本类型 值保存在变量中
int i = 1;
// 引用类型 引用对象地址
String str = "abc";
```

赋值运算符 =
```java
// 基本类型，直接改变变量的值
i = 2;
// 引用类型 指向新的对象
str = "efg";
```

### 1.1.1 例子
```java
// 值传递，num 值不发生变化。
void foo(int val) {
    val = 100;
}
foo(num);

//没有提供改变自身的方法
void foo(String text) {
    //方法接收参数相当于 String text = str;
    //此时 text 指向了一个新的对象，对象的值是"HelloWorld"。
    // str 原对象不发生变化。
    text = "HelloWorld";
}
foo(str);

// 提供了对象改变的方法
void foo(StringBuilder builder) {
    builder.append("a");
}
foo(builder);
```
