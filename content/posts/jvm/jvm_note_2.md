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

3.1 常量：

当方法区的常量不再被任何一个对象引用时

不再被使用的类型：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。

- 加载该类的类加载器已经被回收。

- 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。


## 怎么回收
1.收集

2.清理


### 几种垃圾回收算法
- 分代收集
- 标记清除
- 标记复制
- 标记整理

#### 分代收集
- 弱分代假说：绝大多数对象都是朝生夕灭的。

- 强分代假说：熬过越多次垃圾收集过程的对象就越难以消亡。

- 跨代引用假说：跨代引用相对于同代引用来说仅占极少数。

收集器应该将Java堆划分出不同的区域，然后将回收对象依据其年龄（年龄即对象熬过垃圾收集过程的次数）分配到不同的区域之中存储。
每次回收时只关注如何保留少量存活而不是去标记那些大量将要被回收的对象。

把老年代划分成若干小块，标识出老年代的哪一块内存会存在跨代引用。此后当发生Minor GC时，只有包含了跨代引用的小块内存里的对象才会被加入到GC Roots进行扫描。

#### 标记清除算法
首先标记出所有需要回收的对象，在标记完成后，统一回收掉所有被标记的对象，也可以反过来，标记存活的对象，统一回收所有未被标记的对象。
![](https://i.loli.net/2020/10/17/bQtCuicBNw1mo9F.png)
#### 标记复制算法
为了解决标记清除算法面对大量可回收对象时执行效率低的问题
将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。
![](https://i.loli.net/2020/10/17/KGg7wFDuBX453fc.png)
#### 标记整理
标记-复制算法在对象存活率较高时就要进行较多的复制操作，效率将会降低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。

标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向内存空间一端移动，然后直接清理掉边界以外的内存。
![](https://i.loli.net/2020/10/17/rHQ6Xholmq5O7RN.png)

### 收集算法

#### 1.引用计数法（Reference Counting）

JVM 并没有采用大多数人所说的引用计数法，**是从来没有**。

##### 概念
在对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加一；当引用失效时，计数器值就减一；任何时刻计数器为零的对象就是不可能再被使用的。

##### 优点
原理简单，判定效率也很高

##### 缺陷
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

#### 2.根节点枚举
![](https://img-blog.csdnimg.cn/20200103191432796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzM3MzEzNjU3,size_16,color_FFFFFF,t_70)
使用OopMap存放存放对象的引用地址

##### 安全点
OopMap存放在特定位置，这个位置被称为安全点，当代码执行到安全点时停顿下来进行标记
##### 安全区域
安全区域是指能够确保在某一段代码片段之中，引用关系不会发生变化，因此，在这个区域中任意地方开始垃圾收集都很安全。
##### 记忆集与卡表
为解决对象跨代引用所带来的问题，垃圾收集器在新生代中建立了名为记忆集，记忆集是一种用于记录从非收集区域指向收集区域的指针集合的抽象数据结构。
在实现记忆集的时候，便可以选择更为粗犷的记录粒度来节省记忆集的存储和维护成本
- 字长精度： 每个记录精确到一个机器字长（就是处理器的寻址位数，如常见的32位或64位，这个精度决定了机器访问物理内存地址的指针长度），该字包含跨代指针。

- 对象精度： 每个记录精确到一个对象，该对象里有字段含有跨代指针。

- 卡精度： 每个记录精确到一块内存区域，该区域内有对象含有跨代指针。

其中卡精度所指的是用一种称为“卡表”的方式去实现记忆集 

##### 写屏障
用于维护记忆集

写屏障可以看作在虚拟机层面对“引用类型字段赋值”这个动作的AOP切面，在引用对象赋值时会产生一个环形通知，供程序执行额外的动作，也就是说赋值的前后都在写屏障的覆盖范畴内。
应用写屏障后，虚拟机就会为所有赋值操作生成相应的指令，一旦收集器在写屏障中增加了更新卡表操作，无论更新的是不是老年代对新生代对象的引用，都会执行维护记忆集的操作。

### 垃圾收集器
![WX20201017-201727.png](https://i.loli.net/2020/10/17/2Iy49MrC8ULVd6h.png)

#### Serial 收集器
- 新生代
- 标记复制
- 单线程
- 它进行垃圾收集时，必须暂停其他所有工作线程，直到它收集结束。

![QQ20201017-202102.png](https://i.loli.net/2020/10/17/XPhj68r5DTcqNtZ.png)

#### ParNew 收集器
- 新生代
- 标记复制
- Serial收集器的多线程并行版本
- 在单核心处理器的环境中绝对不会有比Serial收集器更好的效果，甚至由于存在线程交互的开销

![QQ20201017-202459.png](https://i.loli.net/2020/10/17/Kn6PJYxX3BOyZ15.png)

#### Parallel Scavenge 收集器
- 新生代
- 标记复制
- 多线程
- 达到一个可控制的吞吐量

吞吐量：

吞吐量 = 运行用户代码的时间 / (运行用户代码的时间 + 运行垃圾收集器的时间)

如果虚拟机完成某个任务，用户代码加上垃圾收集总共耗费了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。停顿时间越短就越适合需要与用户交互或需要保证服务响应质量的程序，良好的响应速度能提升用户体验；而高吞吐量则可以最高效率地利用处理器资源，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的分析任务。

#### Serial Old 收集器
- Serial收集器的老年代版本
- 单线程
- 标记整理

![QQ20201017-203219.png](https://i.loli.net/2020/10/17/PlijrZNBUp4oVDz.png)

#### Parallel Old 收集器
- Parallel Scavenge 老年代版本
- 多线程
- 标记整理
- 如果新生代选择了Parallel Scavenge收集器，老年代除了Serial Old 收集器以外别无选择

#### CMS 收集器
- 以获取最短回收停顿时间为目标的收集器
- 标记清除
- 占用计算机计算能力导致程序变慢

1）初始标记

其中初始标记、重新标记这两个步骤仍然需要暂停用户线程,初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快；

2）并发标记

并发标记阶段就是从GC Roots的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行；

3）重新标记

重新标记阶段则是为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也远比并发标记阶段的时间短；

4）并发清除

并发清除阶段，清理删除掉标记阶段判断的已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

![QQ20201017-204240.png](https://i.loli.net/2020/10/17/BqwHbhIz4g13fML.png)

#### Garbage First(G1) 收集器
- 程碑式
- 面向堆内存任何部分来组成回收集
- Region的堆内存布局
  
基于Region的内存布局

- G1不再坚持固定大小以及固定数量的分代区域划分，而是把连续的Java堆划分为多个大小相等的独立区域，每一个Region都可以根据需要，扮演新生代的Eden空间、Survivor空间，或者老年代空间。收集器能够对扮演不同角色的Region采用不同的策略去处理，这样无论是新创建的对象还是已经存活了一段时间、熬过多次收集的旧对象都能获取很好的收集效果。

- Region中还有一类特殊的Humongous区域，专门用来存储大对象。G1认为只要大小超过了一个Region容量一半的对象即可判定为大对象。

- 虽然G1仍然保留新生代和老年代的概念，但新生代和老年代不再是固定的了，它们都是一系列区域（不需要连续）的动态集合。G1收集器之所以能建立可预测的停顿时间模型，是因为它将Region作为单次回收的最小单元，即每次收集到的内存空间都是Region大小的整数倍，这样可以有计划地避免在整个Java堆中进行全区域的垃圾收集。


Mixed GC 模式

衡量标准不再是它属于哪个分代，而是哪块内存中存放的垃圾数量最多，回收收益最大，这就是G1收集器的Mixed GC模式。

1）初始标记

仅仅只是标记一下GC Roots能直接关联到的对象，这个阶段需要停顿线程，但耗时很短，而且是借用进行Minor GC的时候同步完成的，所以G1收集器在这个阶段实际并没有额外的停顿。

2）并发标记

从GC Root开始对堆中对象进行可达性分析，递归扫描整个堆里的对象图，找出要回收的对象，这阶段耗时较长，但可与用户程序并发执行。当对象图扫描完成以后，还要重新处理SATB记录下的在并发时有引用变动的对象。

3）最终标记

对用户线程做另一个短暂的暂停，用于处理并发阶段结束后仍遗留下来的最后那少量的SATB记录。

4）筛选回收

负责更新Region的统计数据，对各个Region的回收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划，可以自由选择任意多个Region构成回收集，然后把决定回收的那一部分Region的存活对象复制到空的Region中，再清理掉整个旧Region的全部空间。这里的操作涉及存活对象的移动，是必须暂停用户线程，由多条收集器线程并行完成的。

![QQ20201017-205037.png](https://i.loli.net/2020/10/17/Jzvk2cS4Wf1R5QY.png)