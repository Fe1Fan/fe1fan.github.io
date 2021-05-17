---
title: "JVM 笔记3：Class文件结构"
date: 2020-10-07T14:23:52+08:00
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

将Main.java编译成class文件（具体编译过程-[编译原理](https://fe1.fan//tags/compilation-principle/)）

---

### 1.2 魔数（Magic Number）
> 每个Class文件的头4个字节被称为魔数

魔数的作用是确定这个文件能否被虚拟机接受。Java的魔数为`0xCAFEBABE`，紧接着魔数(0x00000000-0x00000003)的4个字节(0x00000004-0x00000007)为Class的版本号 4、5为次版本号（Minor Version），6、7为主版本号（Major Version）


以Main.class为例
![image.png](https://i.loli.net/2021/04/25/qcwVNkyDsmU3auS.png)

可以看到，魔数为 `0xCAFEBABE`，紧跟着次版本号为 `0x0000`，主版本号为 `0x0037`，十进制的55，即为JDK11。

![Image00114.jpg](https://i.loli.net/2021/04/26/ifLXanNjJ38TA9l.jpg)

---

### 1.3 常量池
> 用于存储关于类、方法、接口等常量的地方

示例 Main.class 的常量池开始位置如图所示

![image.png](https://i.loli.net/2021/04/26/IunVUWPafH7k8sD.png)

其中 第8、9位为u2类型的数据，常量池的长度， 即常量池容量计数值（constant_pool_count），是在编译时期确定下来的。
**常量池计数从1开始**，上图 `0x001D` 为十进制的29，这就代表常量池中有28个常量。

常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）。

- 被模块导出或开放的包
- 类和接口全限定名
- 字段的名称和描述符
- 方法的名称和描述符
- 方法句柄和方法类型
- 动态调用点和动态常量

不同于C/C++，JVM是在加载类的时候进行动态链接的，也就是说这些字段和方法符号在运行时转换后才能获取到真正的内存地址。
  
常量池中每一项常量都是一个表，以下为所有的表结构数据。

![Image00116.jpg](https://i.loli.net/2021/04/26/KrUzpTyqWu7NfSG.jpg)
![Image00117.jpg](https://i.loli.net/2021/04/26/nFPXpekoWKJI52f.jpg)

如图:

![image.png](https://i.loli.net/2021/05/16/bkCHhB9OTeFEgVQ.png)

常量池的第一个位置 `constant_pool[1]`

其中 `0A` TAG 对应上表中10号位置 `CONSTANT_Methodref` 类方法中的符号引用。`CONSTANT_Methodref` 的数据结构如下：


|  类型   | 名称  | 数量 |
|  ----  | ----  | ----|
| u1  | tag |   1  |
| u2  | class_index |   1  |
| u2  | name_and_type_index |   1  |

所以`constant_pool[1]` 对应的位置是 0x0000000A - 0x0000000E 为 `0A 00 06 00 0F`

`0A`为TAG位，用来区分常量池的类型。

然后紧接着 `00 06` 对应着 class_index，指向声明方法的类描述符`CONSTANT_Class_info` 的索引项。该值指向了常量池中的第六个位置，也就是 `constant_pool[6]`,
来看下 `constant_pool[6]`的结构:

![image.png](https://i.loli.net/2021/05/16/O8WghaqVIAeZB7p.png)

`07` 对应着 `CONSTANT_Class_info` 类型，`CONSTANT_Class_info` 的数据结构如下：

|  类型   | 名称  | 数量 |
|  ----  | ----  | ----|
| u1  | tag |   1  |
| u2  | name_index |   1  |

tag依旧是用来区分类型， name_index 是常量池的索引值，它指向常量池中一个 `CONSTANT_Utf8_info` 类型常量，此常量代表了这个类（或者接口）的全限定名。

如图所示 name_index指向了22，`constant_pool[22]`的值如下图：

![image.png](https://i.loli.net/2021/05/16/VAoh1vqsd9YI7NJ.png)

`01` 对应的类型为 `CONSTANT_Utf8`，数据结构如下：

|  类型   | 名称  | 数量 |
|  ----  | ----  | ----|
| u1  | tag |   1  |
| u2  | length |   1  |
| u1  | bytes[] |   length  |

length值说明了这个UTF-8编码的字符串长度是多少字节，它后面紧跟着的长度为length字节的连续数据是一个使用UTF-8缩略编码表示的字符串。

[comment]: <> (> “UTF-8缩略编码与普通UTF-8编码的区别是：从'\u0001'到'\u007f'之间的字符（相当于1～127的ASCII码）的缩略编码使用一个字节表示，从'\u0080'到'\u07ff'之间的所有字符的缩略编码用两个字节表示，从'\u0800'开始到'\uffff'之间的所有字符的缩略编码就按照普通UTF-8编码规则使用三个字节表示。”)

我们看下如图所以的字符串为 `java/lang/Object`

回到最开始的 `CONSTANT_Methodref` 后面的 `00 0F` 对应着 name_and_type_index，指向名称及类型描述符`NameAndType` 的索引项。
`00 0F`对应着15，也就是 `constant_pool[15]`，我们来看下它所对应的位置的数据：

![image.png](https://i.loli.net/2021/05/16/qlncDkR9tzBumh7.png)

`0C` 对应表中的12，类型为：`CONSTANT_NameAndType`，它的数据结构如下：

|  类型   | 名称  | 数量 |
|  ----  | ----  | ----|
| u1  | tag |   1  |
| u2  | name_index |   1  |
| u2  | descriptor_index |   1  |

name_index、descriptor_index分为对应常量池的`constant_pool[7]`和`constant_pool[8]`，分别是 `CONSTANT_Utf8` 类型的 `<init>` 和 `CONSTANT_Utf8` 类型的 `()V`

拼接起来，常量池第一位为：`java/lang/Object."<init>":()V`

经上分析，常量池翻译方法大概如此，下面我们借助 javap 工具机器翻译:
```shell
javap -verbose Main 
```
结果如下：
```shell
Classfile /Users/xxx/Project/Java/blog/src/fan/fe1/hello/Main.class
  Last modified 2021年5月13日; size 426 bytes
  MD5 checksum 24800bad6df89e3f9ae48afdf82181ca
  Compiled from "Main.java"
public class fan.fe1.hello.Main
  minor version: 0
  major version: 55
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #5                          // fan/fe1/hello/Main
  super_class: #6                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            // HelloWorld
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // fan/fe1/hello/Main
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Main.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = Class              #23            // java/lang/System
  #17 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
  #18 = Utf8               HelloWorld
  #19 = Class              #26            // java/io/PrintStream
  #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
  #21 = Utf8               fan/fe1/hello/Main
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
{
  public fan.fe1.hello.Main();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 4: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String HelloWorld
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8
}
SourceFile: "Main.java"
```

其中常量池部分的28个常量已经帮我们计算好：
```shell
Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            // HelloWorld
   #4 = Methodref          #19.#20        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #21            // fan/fe1/hello/Main
   #6 = Class              #22            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               SourceFile
  #14 = Utf8               Main.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = Class              #23            // java/lang/System
  #17 = NameAndType        #24:#25        // out:Ljava/io/PrintStream;
  #18 = Utf8               HelloWorld
  #19 = Class              #26            // java/io/PrintStream
  #20 = NameAndType        #27:#28        // println:(Ljava/lang/String;)V
  #21 = Utf8               fan/fe1/hello/Main
  #22 = Utf8               java/lang/Object
  #23 = Utf8               java/lang/System
  #24 = Utf8               out
  #25 = Utf8               Ljava/io/PrintStream;
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
```
跟我们手动计算的是一样的。

剩下的常量的类型结构如下图：

![1.jpg](https://i.loli.net/2021/05/16/nqfBJ6wCSFKT4Qa.jpg)
![2.jpg](https://i.loli.net/2021/05/16/9LCHXig5UjdTGQz.jpg)
![3.jpg](https://i.loli.net/2021/05/16/dFDS2ajvWkwpnNJ.jpg)

---

### 1.4 访问标志
入下图所示：

![image.png](https://i.loli.net/2021/05/16/5MNFLC6xnTleOAs.png)

紧跟在常量池后面的2个字节代表了访问标识 `access_flags` ，这个标志用户识别一些类或者接口层次的访问信息，包括 这个Class是类还是接口，是否定义为public，是否定义为abstract，如果是类的话，是否被生命为final等等。

具体的访问标识如下：

![Image00124.jpg](https://i.loli.net/2021/05/16/pBcv2N1gas6SUmC.jpg)

本类的 `access_flags` 为 0x0001 | 0x0020 = 0x0021(ACC_PUBLIC, ACC_SUPER)

---

### 1.5 类索引、父类索引与接口索引集合

- 类索引   (u2)
- 父类索引 (u2)
- 索引集合 ([]u2)

Class文件由这三项数据来确定该类型的继承关系，类索引用于确定累的权限定名，父类索引用于确定这个类的父类的权限定名，Java不支持多继承，所以父类索引只有一个，除 java.lang.Object 之外的所有Java类都有父类，
因此除了 java.lang.Object 外，所有的父类索引都不为0。接口索引结合用来秒速这个类实现了哪些接口，这些被实现的接口将按 implements 后的接口顺序从左导游排列在接口集合中。

类索引和父类索引各自指向一个类型为 `CONSTANT_Class_info` 的类描述符常量，通过类描述符常量来定位到 `CONSTANT_Utf8_info` 中的权限定名字字符串。

对于接口索引集合，入口用u2类型表示接口计数器 `u2 interfaces_count`，如果计数器为0，后面的接口索引表则不占用任何字符。以开始代码为例：

```shell
  this_class: #5                          // fan/fe1/hello/Main
  super_class: #6                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
```

---

### 1.6 字段表集合

> 字段表(field_info) 用户描述接口或者类中声明的变量。包括类级变量和实例级变量，但不包括方法内部的局部变量。

字段表的结构如下：

|  类型   | 名称  | 数量 |
|  ----  | ----  | ----|
| u2  | access_flags |   1  |
| u2  | name_index |   1  |
| u2  | descriptor_index |   1  |
| u2  | attributes_count |   1  |
| attribute_info  | attributes |   attributes_count  |

访问标识符：

![Image00129.jpg](https://i.loli.net/2021/05/16/PVgmpxaGuo59liC.jpg)

紧跟着访问标识符的是 name_index和descriptor_index，它们都是对常量池子项的引用，分别代表字段的简单名称以及字段和方法的描述符。

- 全限定名

"fan/fe1/hello/Main"
仅仅是把类全名中的"."替换成了"/"，
为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个";"号表示全限定名结束。


- 简单名称

简单名称则就是指没有类型和参数修饰的方法或者字段名称。

- 方法和字段的描述符

作用是用来描述字段的数据类型、方法的参数列表（数量、类型及顺序）和返回值。

基本数据类型 (byte、char、double、float、int、long、short、boolean)以及void类型都是用大写字符来表示，而对象类型则用字符L加对象的全限定名表示。


|  标识字符   | 含义  | 
|  ----  | ----  | 
|B| byte|
|C| char|
|D| double|
|F| float|
|I| int|
|J| long|
|S| short|
|Z| boolean|
|V| void|
|L| 对象 例如: Ljava/lang/Object;|

对于数组来说，每一个纬度将使用一个前置的 `[` 来标识，例如 `String[][]` 类型的二维数组将被记录为 `[[Ljava/lang/String;`，一个整数数组 `int[]` 将被记录为 `[I`。

用描述符描述方法时，按照先参数列表、后返回值的顺序，方法toString() 被描述为 `()Ljava/lang/String;`，方法
```java

int indexOf(char[] source, int sourceOffset, int sourceCount, char[] target, int targetOffset, int targetCount, int fromIndex)

```
的描述符为: `([CII[CIII)I`。

我们把demo代码改成：

```java
package fan.fe1.hello;


public class Main {

    private static String name = "HelloWorld";
    public static void main(String[] args) {
        System.out.println(name);
    }
}
```
这时候再看javap的结果：
```shell

Classfile /Users/xxx/Project/Java/blog/src/fan/fe1/hello/Main.class
  Last modified 2021年5月16日; size 528 bytes
  MD5 checksum 2339f230b8849c9268bdb490e6e67e42
  Compiled from "Main.java"
public class fan.fe1.hello.Main
  minor version: 0
  major version: 55
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #6                          // fan/fe1/hello/Main
  super_class: #7                         // java/lang/Object
  interfaces: 0, fields: 1, methods: 3, attributes: 1
Constant pool:
   #1 = Methodref          #7.#19         // java/lang/Object."<init>":()V
   #2 = Fieldref           #20.#21        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Fieldref           #6.#22         // fan/fe1/hello/Main.name:Ljava/lang/String;
   #4 = Methodref          #23.#24        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = String             #25            // HelloWorld
   #6 = Class              #26            // fan/fe1/hello/Main
   #7 = Class              #27            // java/lang/Object
   #8 = Utf8               name
   #9 = Utf8               Ljava/lang/String;
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               <clinit>
  #17 = Utf8               SourceFile
  #18 = Utf8               Main.java
  #19 = NameAndType        #10:#11        // "<init>":()V
  #20 = Class              #28            // java/lang/System
  #21 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
  #22 = NameAndType        #8:#9          // name:Ljava/lang/String;
  #23 = Class              #31            // java/io/PrintStream
  #24 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
  #25 = Utf8               HelloWorld
  #26 = Utf8               fan/fe1/hello/Main
  #27 = Utf8               java/lang/Object
  #28 = Utf8               java/lang/System
  #29 = Utf8               out
  #30 = Utf8               Ljava/io/PrintStream;
  #31 = Utf8               java/io/PrintStream
  #32 = Utf8               println
  #33 = Utf8               (Ljava/lang/String;)V
{
  public fan.fe1.hello.Main();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 4: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: getstatic     #3                  // Field name:Ljava/lang/String;
         6: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         9: return
      LineNumberTable:
        line 8: 0
        line 9: 9

  static {};
    descriptor: ()V
    flags: (0x0008) ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: ldc           #5                  // String HelloWorld
         2: putstatic     #3                  // Field name:Ljava/lang/String;
         5: return
      LineNumberTable:
        line 6: 0
}
SourceFile: "Main.java"
```

其中 interfaces 为：

```shell
interfaces: 0, fields: 1, methods: 3, attributes: 1
```

再找到字节码部分：

![image.png](https://i.loli.net/2021/05/16/mDwjYhs85Bk9Ket.png)

其中 `00 01` 代表有一个字段表数据。接下来紧跟着 `struct field_info fields` 类型为

| 名称  | 值 | 0x |
|----|----|----|
|access_flags|10| 00 0A|
| name_index |   8  | 00 08 |
| descriptor_index |   9  | 00 09 |
| attributes_count |   0  | 00 00 |

其中 access_flags 为 0x0002 | 0x0008  = 0x0000A

name_index 为 0x0008，指向 `constant_pool[8]`，为 `CONSTANT_Utf8` 的 name

descriptor 为 0x0009，指向 `constant_pool[9]`，为 `CONSTANT_Utf8` 的 Ljava/lang/String;

attributes_count 为 0x0000，代表属性表计数器为0。

如果我们将字段改成final，那就可能存在一项为 ConstantValue 的属性，其值指向常量 HelloWorld。

---

### 1.7 方法表集合
> Class文件存储格式中对方法的描述

方法表集合的结构如下：

| 类型  | 名称 | 数量 |
 |----|----|----|
 |u2|access_flags|1|
 |u2|name_index|1|
 |u2|descriptor_index|1|
 |u2|attributes_count|1|
| attribute_info  | attributes |   attributes_count  |

访问符如下图：

![image.png](https://i.loli.net/2021/05/16/hp7Yax1AOfMeBwd.png)

跟之前一样，入口为methods_count，是方法计数器，如图为 `0x0003`，代表有3个方法表。

在 methods_count 后紧跟着的第一个 methods[0]：

| 名称  | 值 | 0x |
|----|----|----|
|access_flags|1|00 01|
|name_index|10|00 0A|
|descriptor_index|11|00 0B|
|attributes_count|1|00 01|


根据指向地址计算出 methods[0]为 <init>()V

根据如上 分别计算出 methods[1]、methods[2] 分别为 `main([Ljava/lang/String;)V` 、`<clinit>()V`。

在Java中，要重载一个方法除了要求具有相同的简单名称之外，还要求必须拥有一个与原方法不同的特征签名。特征签名是指
一个方法中各个参数在常量池重的字段符号引用的集合。也正是因为返回值不会包含在特征签名之中，所以Java语言里面是无法仅仅依靠返回值的不同来对一个已有方法进行重载的。但是在Class文件格式之中，特征签名的范围明显要更大一些，只要描述符不是完全一致的两个方法就可以共存。也就是说，如果两个方法有相同的名称和特征签名，但返回值不同，那么也是可以合法共存于同一个Class文件中的。

---

### 1.8 属性表集合
> Class文件、字段表、方法表都可以携带自己的属性表集合，以描述某些场景专有的信息。

属性如下图：

![1.jpg](https://i.loli.net/2021/05/16/XaI9wysoH3pRqV4.jpg)
![2.jpg](https://i.loli.net/2021/05/16/B1sleXZWFx75aqL.jpg)

#### Code
> Java程序方法体里面的代码经过Javac编译器处理之后，最终变为字节码指令存储在Code属性内。

结构如下：



| 类型  | 名称 | 数量 |
|----|----|----|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2 |max_stack|1|
|u2 | max_locals|1|
|u4 | code_length|1|
|u1|code|code_length| 
|u2|exception_table_length|1| 
|exception_info|exception_table|exception_table_length| 
|u2|attributes_count|1| 
|attribute_info|attributes|attributes_count|

attribute_name_index 是一项指向CONSTANT_Utf8_info型常量的索引，此常量值固定为"Code"。

attribute_length 指示了属性值的长度，由于属性名称索引与属性长度一共为6个字节，所以属性值的长度固定为整个属性表长度减去6个字节。

max_stack 代表了操作数栈深度的最大值，JVM根据这个值来分配栈帧重的操作栈深度。

max_locals 代表了局部变量所需要的存储空间，单位是变量槽（Slot），变量槽是虚拟机为内部变量分配内存所使用的最小单位。
byte、char、float、int、short、boolean和returnAddress等长度不超过32位的数据类型，每个局部变量占用一个变量槽，而double和long这两种64位的数据类型则需要两个变量槽来存放。

方法参数（包括实例方法中的隐藏参数“this”）、显式异常处理程序的参数（Exception Handler Parameter，就是try-catch语句中catch块中所定义的异常）、方法体中定义的局部变量都需要依赖局部变量表来存放。注意，并不是在方法中用了多少个局部变量，就把这些局部变量所占变量槽数量之和作为max_locals的值，操作数栈和局部变量表直接决定一个该方法的栈帧所耗费的内存，不必要的操作数栈深度和变量槽数量会造成内存的浪费。Java虚拟机的做法是将局部变量表中的变量槽进行重用，当代码执行超出一个局部变量的作用域时，这个局部变量所占的变量槽可以被其他局部变量所使用，Javac编译器会根据变量的作用域来分配变量槽给各个变量使用，根据同时生存的最大局部变量数量和类型计算出max_locals的大小。

code_length 和 code 用来存储Java远程与编译后的字节码，code_length代表字节码长度、code用于春初字节码指令的一系列字节流。
每个指令就是一个u1类型的单字节，当JVM读取到code中的一个字节码时，就可以找到对应的指向另，并且知道指令是否需要参数，以及如何解析。

code_length虽然是u4类型，理论长度为2的32次方，但是JVM规范中限制一个方法不允许超过65535条字节码指令。

假如把Java程序中的信息分为代码和元数据(Metadata，包括类、字段、方法定义及其它信息)两部分，那么Code属性用于描述代码、其他数据项目都用于描述元数据。

在1.7章节中，分析 methods[0] 时，attributes_count 为1，那么意味着这个方法带有自己的属性表集合，attribute_info：

| 名称  | 值 | 0x |
|----|----|----|
|attribute_name_index | 12 |00 0C|
|attribute_length | 29 |00 00 00 1D|
|max_stack | 1 |00 01|
|max_locals | 1 |00 01|
|code_length | 5 |00 00 00 05|
|code||2A B7 00 01 B1|
|exception_table_length | 0 |00 00|
|attributes_count | 1 |00 01|
|attribute_info |  |00 0D 00 00 00 06 00 01 00 00 00 04|

其中 code 为 `2A B7 00 01 B1`，当JVM读取翻译 code 时：

- 读取 2A 对应 aload_0 将第0个变量槽中为reference类型的本地变量推送到操作数栈顶。
- 读取 B7 对应 invokespecial 以栈顶的reference类型的数据所指向的对象作为方法的接收者，掉哟哦那个此对象的实例构造器方法、private方法或者其它父类的方法
这个方法有一个u2类型的参数说明调用哪一个方法，它指向常量池重的一个CONSTANT_Methodref_info类型常量，即此方法的符号引用。
- 读取 0001，这是invokespecial指令的参数，代表一个符号引用，查常量池0x0001对应的常量为实例的构造器方法的符号引用。
- 读入 B1 对应return 从方法的返回，并且返回值为void。

我们截取javap代码部分如下：
```java
{
  public fan.fe1.hello.Main();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 4: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: getstatic     #3                  // Field name:Ljava/lang/String;
         6: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         9: return
      LineNumberTable:
        line 8: 0
        line 9: 9

  static {};
    descriptor: ()V
    flags: (0x0008) ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: ldc           #5                  // String HelloWorld
         2: putstatic     #3                  // Field name:Ljava/lang/String;
         5: return
      LineNumberTable:
        line 6: 0
}

```

其中 init 中存在 args_size = 1，locals = 1，是因为this关键字，编译器编译的时候把对this关键字的访问转变成对一个普通方法参数的访问，然后在虚拟机调用实例方法
的时候传入此参数而已，因此在实例方法的局部变量表中至少会存在一个指向当前对象实例的局部变量，局部变量表也会预留出第一个变量槽位来春芳对象实例的引用，所以实例方法参数值从1
开始计算。

紧跟在code后面的 是异常表的长度，当异常存在时，紧跟在 exception_table_length 后面的结构为：

| 类型  | 名称 | 数量 |
|----|----|----|
|u2|start_pc|1|
|u2|end_pc|1|
|u2|handler_pc|1|
|u2|catch_type|1|

结构的意思是：
当从 start_pc 行 到 end_pc 行之间（不包含end_pc行）出现了类型为 catch_type 或则其子类型的异常
则跳转到 handler_pc 行继续处理，当catch_type为0时，代表任意的异常情况都需要转到 handler_pc 行处理。

我们修改代码为：
```java
package fan.fe1.hello;


public class Main {

    private static int value = 1;


    public static void main(String[] args) {
        try {
            System.out.println(value);
        } catch (Exception e) {
            value += 1;
            System.out.println(value);
        } finally {
            value += 2;
            System.out.println(value);
        }
    }
}
```

现在我们通过javap查看main方法的代码部分：

```shell
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: getstatic     #3                  // Field value:I
         6: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
         9: getstatic     #3                  // Field value:I
        12: iconst_2
        13: iadd
        14: putstatic     #3                  // Field value:I
        17: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        20: getstatic     #3                  // Field value:I
        23: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
        26: goto          87
        29: astore_1
        30: getstatic     #3                  // Field value:I
        33: iconst_1
        34: iadd
        35: putstatic     #3                  // Field value:I
        38: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        41: getstatic     #3                  // Field value:I
        44: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
        47: getstatic     #3                  // Field value:I
        50: iconst_2
        51: iadd
        52: putstatic     #3                  // Field value:I
        55: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        58: getstatic     #3                  // Field value:I
        61: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
        64: goto          87
        67: astore_2
        68: getstatic     #3                  // Field value:I
        71: iconst_2
        72: iadd
        73: putstatic     #3                  // Field value:I
        76: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        79: getstatic     #3                  // Field value:I
        82: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
        85: aload_2
        86: athrow
        87: return
      Exception table:
         from    to  target type
             0     9    29   Class java/lang/Exception
             0     9    67   any
            29    47    67   any
      LineNumberTable:
        line 11: 0
        line 16: 9
        line 17: 17
        line 18: 26
        line 12: 29
        line 13: 30
        line 14: 38
        line 16: 47
        line 17: 55
        line 18: 64
        line 16: 67
        line 17: 76
        line 18: 85
        line 19: 87
      StackMapTable: number_of_entries = 3
        frame_type = 93 /* same_locals_1_stack_item */
          stack = [ class java/lang/Exception ]
        frame_type = 101 /* same_locals_1_stack_item */
          stack = [ class java/lang/Throwable ]
        frame_type = 19 /* same */

```

在此代码块中，存在三个异常table，分别是 `0-9->29` `0-9->67` `29-47->67`。

先来看第一段 `from 0 to 9 target 29`:

```shell
0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
3: getstatic     #3                  // Field value:I
6: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
...
29: astore_1
30: getstatic     #3                  // Field value:I
33: iconst_1
34: iadd
35: putstatic     #3                  // Field value:I
38: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
41: getstatic     #3                  // Field value:I
44: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
47: getstatic     #3                  // Field value:I
50: iconst_2
51: iadd
52: putstatic     #3                  // Field value:I
55: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
58: getstatic     #3                  // Field value:I
61: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
64: goto          87                  // Return
```
其中 0-6 为正常的输出，如果出现异常 `Class java/lang/Exception` 则跳转到29，
然后再看29-64，其中 29-44 为：
```shell
29: astore_1
30: getstatic     #3                  // Field value:I
33: iconst_1
34: iadd
35: putstatic     #3                  // Field value:I
38: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
41: getstatic     #3                  // Field value:I
44: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
```

是catch中对变量进行+操作的并输出的操作。

其中 47-64 为：
```shell
47: getstatic     #3                  // Field value:I
50: iconst_2
51: iadd
52: putstatic     #3                  // Field value:I
55: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
58: getstatic     #3                  // Field value:I
61: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
64: goto          87                  // Return
```

是 finally 中对变量进行+操作并输出的操作。合并起来的流程就是： try->catch->finally，前提条件就是拦截到的异常是catch的异常或其子类。

下面再看第二个 `0-9->67`：

```shell
0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
3: getstatic     #3                  // Field value:I
6: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
...
67: astore_2
68: getstatic     #3                  // Field value:I
71: iconst_2
72: iadd
73: putstatic     #3                  // Field value:I
76: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
79: getstatic     #3                  // Field value:I
82: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
85: aload_2
86: athrow
87: return 
```
0-6依旧是正常的输出，那么67则为出去catch异常的异常发生时，走的finally代码块。

接着再看第三个 `29-47->67`:

```shell
29: astore_1
30: getstatic     #3                  // Field value:I
33: iconst_1
34: iadd
35: putstatic     #3                  // Field value:I
38: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
41: getstatic     #3                  // Field value:I
44: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
...
67: astore_2
68: getstatic     #3                  // Field value:I
71: iconst_2
72: iadd
73: putstatic     #3                  // Field value:I
76: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
79: getstatic     #3                  // Field value:I
82: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
85: aload_2
86: athrow
87: return 
```
上文说过 27-44为catch中的代码块，如果catch发生任何异常，则转到finally块。

总结一下异常：
- 如果try出现异常，异常属于catch，则跳转catch
- 如果try出现异常，异常不属于catch，则跳转finally
- 如果catch出现任何异常，则跳转finally

这里就引申出来一个新的问题，try 和 catch 里吗存在return 时，finally代码块是否会执行，什么时候执行，如果有return是否会覆盖上面的return ？

我们新增一个result方法：
```java
public int result() {
        try {
            value ++;
            System.out.println("try");
            return value;
        } catch (Exception e) {
            value ++;
            System.out.println("catch");
            return value;
        } finally {
            value ++;
            System.out.println("finally");
            return value;
        }
    }
```

查看编译后的字节码:

```shell
  public int result();
    descriptor: ()I
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
         0: getstatic     #2                  // Field value:I
         3: iconst_1
         4: iadd
         5: putstatic     #2                  // Field value:I
         8: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: ldc           #4                  // String try
        13: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        16: getstatic     #2                  // Field value:I
        19: istore_1
        20: getstatic     #2                  // Field value:I
        23: iconst_1
        24: iadd
        25: putstatic     #2                  // Field value:I
        28: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        31: ldc           #6                  // String finally
        33: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        36: getstatic     #2                  // Field value:I
        39: ireturn
        40: astore_1
        41: getstatic     #2                  // Field value:I
        44: iconst_1
        45: iadd
        46: putstatic     #2                  // Field value:I
        49: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        52: ldc           #8                  // String catch
        54: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        57: getstatic     #2                  // Field value:I
        60: istore_2
        61: getstatic     #2                  // Field value:I
        64: iconst_1
        65: iadd
        66: putstatic     #2                  // Field value:I
        69: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        72: ldc           #6                  // String finally
        74: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        77: getstatic     #2                  // Field value:I
        80: ireturn
        81: astore_3
        82: getstatic     #2                  // Field value:I
        85: iconst_1
        86: iadd
        87: putstatic     #2                  // Field value:I
        90: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        93: ldc           #6                  // String finally
        95: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        98: getstatic     #2                  // Field value:I
       101: ireturn
      Exception table:
         from    to  target type
             0    20    40   Class java/lang/Exception
             0    20    81   any
            40    61    81   any
      LineNumberTable:
        line 10: 0
        line 11: 8
        line 12: 16
        line 18: 20
        line 19: 28
        line 20: 36
        line 13: 40
        line 14: 41
        line 15: 49
        line 16: 57
        line 18: 61
        line 19: 69
        line 20: 77
        line 18: 81
        line 19: 90
        line 20: 98
      StackMapTable: number_of_entries = 2
        frame_type = 104 /* same_locals_1_stack_item */
          stack = [ class java/lang/Exception ]
        frame_type = 104 /* same_locals_1_stack_item */
          stack = [ class java/lang/Throwable ]

```
由上面字节码分析可知，无论有没有异常，finally的return才会真正的返回。

我们再去掉 finally 里的return：
```java
public int result() {
        try {
            value ++;
            System.out.println("try");
            return value;
        } catch (Exception e) {
            value ++;
            System.out.println("catch");
            return value;
        } finally {
            value ++;
            System.out.println("finally");
        }
    }
```

字节码：
```shell
  public int result();
    descriptor: ()I
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=2, locals=4, args_size=1
         0: getstatic     #2                  // Field value:I
         3: iconst_1
         4: iadd
         5: putstatic     #2                  // Field value:I
         8: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: ldc           #4                  // String try
        13: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        16: getstatic     #2                  // Field value:I
        19: istore_1
        20: getstatic     #2                  // Field value:I
        23: iconst_1
        24: iadd
        25: putstatic     #2                  // Field value:I
        28: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        31: ldc           #6                  // String finally
        33: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        36: iload_1
        37: ireturn
        38: astore_1
        39: getstatic     #2                  // Field value:I
        42: iconst_1
        43: iadd
        44: putstatic     #2                  // Field value:I
        47: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        50: ldc           #8                  // String catch
        52: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        55: getstatic     #2                  // Field value:I
        58: istore_2
        59: getstatic     #2                  // Field value:I
        62: iconst_1
        63: iadd
        64: putstatic     #2                  // Field value:I
        67: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        70: ldc           #6                  // String finally
        72: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        75: iload_2
        76: ireturn
        77: astore_3
        78: getstatic     #2                  // Field value:I
        81: iconst_1
        82: iadd
        83: putstatic     #2                  // Field value:I
        86: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        89: ldc           #6                  // String finally
        91: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        94: aload_3
        95: athrow
      Exception table:
         from    to  target type
             0    20    38   Class java/lang/Exception
             0    20    77   any
            38    59    77   any
      LineNumberTable:
        line 10: 0
        line 11: 8
        line 12: 16
        line 18: 20
        line 19: 28
        line 12: 36
        line 13: 38
        line 14: 39
        line 15: 47
        line 16: 55
        line 18: 59
        line 19: 67
        line 16: 75
        line 18: 77
        line 19: 86
        line 20: 94
      StackMapTable: number_of_entries = 2
        frame_type = 102 /* same_locals_1_stack_item */
          stack = [ class java/lang/Exception ]
        frame_type = 102 /* same_locals_1_stack_item */
          stack = [ class java/lang/Throwable ]

```

由此可见，return是再finally之后才进行返回的。
