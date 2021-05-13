---
title: "JVM 笔记3：虚拟机执行系统"
date: 2020-10-07T14:23:52+08:00
draft: true
tags: [jvm]
---

<!--more--> 
## 1.0 Class 文件结构
> Class文件是一组以8个字节为基础单位的二进制流

Class文件采用由两种数据类型构成的伪结构，分别是"无符号数"和"表"。

- 无符号数 基本的数据类型，u1、u2、u4、u8分别代表1字节、2字节、4字节、8字节，无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。
- 表 是由多个无符号数或者其他表作为数据项构成的复合数据类型，以info结尾，整个Class文件本质上也可以视作是一张表，

![Image00112.jpg](https://i.loli.net/2021/04/25/SClnpOToP8hmvt6.jpg)

### 1.1 开始之前
本文开始以一个名为Main.java的文件进行进行分析
``` java
package fan.fe1.hello;

public class Main {
    public static void main(String[] args) {
        System.out.println("HelloWorld");
    }
}
```

---

### 1.2 魔数（Magic Number）
> 每个Class文件的头4个字节被称为魔数

魔数的作用是确定这个文件能否被虚拟机接受。Java的魔数为`0xCAFEBABE`，紧接着魔数的4个字节(0-3)为Class的版本号 4、5为次版本号（Minor Version），6、7为主版本号（Major Version）


以Main.java为例
![image.png](https://i.loli.net/2021/04/25/qcwVNkyDsmU3auS.png)

可以看到，魔数为 `0xCAFEBABE`，紧跟着次版本号为 `0x0000`，主版本号为 `0x0037`，十进制的55，即为JDK11。

![Image00114.jpg](https://i.loli.net/2021/04/26/ifLXanNjJ38TA9l.jpg)

---

### 1.3 常量池
> 用于存储关于类、方法、接口等常量的地方

示例 Main.class 的常量池开始位置如图所示

![image.png](https://i.loli.net/2021/04/26/IunVUWPafH7k8sD.png)

其中 第8、9位为u2类型的数据，常量池的长度， 即常量池容量计数值（constant_pool_count），是在编译时期确定下来的。
常量池计数从1开始，上图 `0x001D` 为十进制的29，这就代表常量池中有28个常量。

常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）。

- 被模块导出或开放的包
- 类和接口全限定名
- 字段的名称和描述符
- 方法的名称和描述符
- 方法句柄和方法类型
- 动态调用点和动态常量

Class文件中不会保存各个方法、字段最终在内存中的布局信息，在类加载的时候JVM负责把常量布局到真正的内存。
  
常量池中每一项常量都是一个表，以下为所有的表结构数据。

![Image00116.jpg](https://i.loli.net/2021/04/26/KrUzpTyqWu7NfSG.jpg)
![Image00117.jpg](https://i.loli.net/2021/04/26/nFPXpekoWKJI52f.jpg)
