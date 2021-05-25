---
title: "Redis 笔记1：基础"
date: 2020-10-07T14:23:52+08:00
tags: [redis]
math: true
---
<!--more--> 
# 1 基础数据结构

- string
- list
- hash
- set
- zset

## 1.1 string

### 1.1.1 简介
string是Redis中最常用的数据结构，其长度是可变的，采用的内存分配策略是
预分配冗余空间的方式来减少内存的频繁分配，当字符串长度小于 1M 时，扩容都是加倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间。
需要注意的是字符串最大长度为 512M。

### 1.1.2 使用
常用

```shell
set k v
get k v
mset k1 v1 k2 v2 k3 v3
mget k1 k2 k3
```

过期

```shell
#过期
expire k 5  #5秒后过期
setex k 5 v
setnx k 5 v #创建并过期 如果k不存在则创建，如果存在则不变。
```

计数
```shell
set k 10
incr k
> 11
incrby k 5
> 16
incrby k -5
> 11
```

### 1.1.3 底层结构
SDS(Simple Dynamic String)

Redis 使用SDS管理字符串，SDS结构如下：

|名称|说明|
|----|----|
|free|数组中未使用字节的数量|
|len|数组中已使用戒子的数量|
|buf|字节数组，用户保存字符串|

假设使用命令`set k1 Redis`，此时 free为0，len为5，buf分别表存了`R`，`e`，`d`，`i`，`s`，`\0`。

优势：
- 记录了字符串的长度，存储为连续的内存块，操作复杂度为O(1)。
- 防止缓存区溢出。
  
    当两个值位于相邻的两个内存位置时，如果不检测当前修改字符串是否超过原有空间，则会覆盖到后面值的内容。
    所以SDS当对字符串进行修改时，会先检查分配的空间是否满足修改所需要，如果不满足则将SDS空间扩展至所需大小
- 减少修改字符串带来的内存重分配次数。
    - 预分配策略

      在SDS中，buf的数组长度不一定就是字符数量加一，数组中包含未使用的字节，使用free记录。
      当对SDS进行修改时，程序不仅会为SDS分配修改所必要的空间，还会为SDS分配额外的未使用空间。
      其中预分配空间数量由以下公式决定：
        1. 如果对SDS修改后，len小于1MB，那么程序分配和len属性同样大小的未使用空间，这时的len和free相同。
        2. 如果对SDS修改后，len大于等于1MB，那么程序分配1M的未使用空间。
    - 惰性空间释放
      
      用于优化SDS字符串缩短操作，当SDS缩短字符串时，程序并不立即使用内存重新分配来回收多出来的字节，而是使用free记录下来，并等待将来使用。



## 1.2 list

### 1.2.1 简介
Redis中的list是双向链表而不是数组，这意味着插入和删除非常快，但是索引很慢。当列表弹出了最后一个元素之后，该数据结构自动被删除，内存被回收。
因为底层是连续的内存块，所以速度很块。

### 1.2.2 使用
右进左出：队列
```shell
rpush lk v1 v2 v3 v4
llen lk
> 4
lpop lk
> v1
lpop lk
> v2
```

右进右出：栈

```shell
rpush lk v1 v2 v3 v4
llen lk
> 4
lpop lk
> v4
lpop lk
> v3
```

获取值：
```shell
rpush lk v1 v2 v3 v4
llen lk
> 4
lindex lk 1 #查找第一个元素
> v1
lrange lk 0 -1 # 查找所有元素 -1代表倒数第一个元素
> 1) v1
> 2) v2
> 3) v3
> 4) v4
ltrim lk 1 -1   # 保留
lrange lk 0 -1
> v1
> v2
ltrim lk 1 0
llen lk
> 0
```

### 1.2.3 底层结构

双向链表

Node Struct:

|名称|说明|
|----|----|
|prev|前节点指针|
|next|后节点指针|
|value|存储值|

Node List Struct:
|名称|说明|
|----|----|
|head|表头节点|
|tail|表尾节点|
|len|节点数量|
|dup(函数)|节点值复制函数|
|free(函数)|节点值释放函数|
|match(函数)|节点值对比函数|

特点：
- 双向链表，获取前一个和后一个节点复杂度为O(1)。
- 无环，表头节点的prev和表尾节点的next都指向NULL，以NULL为终点。
- 带表头和表尾指针，获取头尾复杂度O(1)。
- 带链表长度计数器，获取数量复杂度O(1)。

## 1.3 hash

### 1.3.1 简介
Redis Hash 使用数组 + 链表二维结构，如果hash碰撞，则放到对应hash的链表中。
Redis的值只能是字符串，

### 1.3.2 使用

```shell
hset hk k1 v1
hset hk k2 v2
hlen hk
> 2
hgetall hk
> k1
> v1
> k2
> v2
hget hk k1
> v1
hset hk k1 v3
hget hk k1
> v3
# 根据单个key计算

```

### 1.3.3 底层结构

哈希表

dict Struct:
|名称|说明|
|----|----|
|type|类型特定函数|
|privdata|私有数据|
|dictht|哈希表，长度为2|
|trehashidx|rehash索引，当rehash不在进行时，值为-1|

type属性和privdata属性是针对不同类型的键值对，为创建多态字典而设置的：

- type属性是一个指向dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数，Redis会为用途不同的字典设置不同的类型特定函数。 
- 而privdata属性则保存了需要传给那些类型特定函数的可选参数。

dictType Struct:
|名称|说明|
|----|----|
|hashFunction(函数)|计算哈希值函数|
|keyDup(函数)|复制键的函数|
|valDup(函数)|复制值的函数|
|keyCompare(函数)|对比键的函数|
|keyDestructor(函数)|销毁键的函数|
|valDestructor(函数)|销毁值的函数|


ht属性是一个包含两个项的数组，数组中的每个项都是一个dictht哈希表，一般情况下，字典只使用ht[0]哈希表，ht[1]哈希表只会在对ht[0]哈希表进行rehash时使用。

dictht Struct:
|名称|说明|
|----|----|
|table|哈希表数组|
|size|哈希表大小|
|sizemask|哈希表大小掩码，用于计算索引值，总是等于size - 1|
|used|改哈希表已有节点数量|

table数组中的每个元素都指向一个dictEntry结构，每个dictEntry结构保存着一个键值对。

dictEntry Struct:
|名称|说明|
|----|----|
|key|键|
|v|值|
|next|指向下个哈希表节点，形成链表|

next属性是指向另一个哈希表节点的指针，这个指针可以将多个哈希值相同的键值对连接在一次，以此来解决键冲突（collision）的问题。


![00372.jpg](https://i.loli.net/2021/05/24/5XpGM9DHdbae7yr.jpg)


当要将一个新的键值对添加到字典里面时，程序需要先根据键值对的键计算出哈希值和索引值，然后再根据索引值，将包含新键值对的哈希表节点放到哈希表数组的指定索引上面。

```c
hash = dict->type->hashFunction(k0);
// 假设hash值为8
index = hash&dict-> ht[0].sizemask = 8&3 = 0;
// 计算出键k0的索引值0，这表示包含键值对k0和v0的节点应该被放置到哈希表数组的索引0位置上
```

![00381.jpg](https://i.loli.net/2021/05/24/2xyt6EvH13uLTSz.jpg)

### 1.3.4 hash冲突
当两个或以上的键被分配到哈希表数组的同一个位置，就是发生了冲突。

redis使用链地址法解决冲突，多个哈希表节点用next指针形成一个单向链表。

在放入链表时，因为没有尾节点的指针，所以新的值总放在链表的表头位置。

### 1.3.5 rehash
当哈希键值对过多或者过少时，程序需要对哈希表进行扩展或收缩。步骤如下：

- 对 ht[1]进行分配空间，空间大小取决于ht[0] 键值对的数量，也就是ht[0].used。
    - 如果是扩展操作，那么ht[1]的大小为`第一个大于等于ht[0].used*2`的2的n次方。
    - 如果是收缩操作，那么ht[1]的大小为``第一个大于ht[0].used的数的`2的n次方。
- 将ht[0]的所有键值对rehash到ht[1]上，rehash指的是重新计算键的哈希值和索引值，然后将键值对放置到ht[1]哈希表的指定位置上。
- 当ht[0]包含的所有键值对都迁移到ht[1]之后，释放ht[0],将ht[1]设置为ht[0]，并在ht[1]创建一个空白的哈希表，为下次rehash做准备。

### 1.3.6 渐进式hash
rehash这个动作并不是一次性的，而是分多次进行，过程如下：

- 为ht[1]分配空间。
- 在字典中维护一个索引计数器变量 rehashidx，并将他的值设为0，标识rehash工作开始。
- 在rehash操作期间，每次对字典执行的添加、删除、查找或者更新操作，还会顺带将ht[0]哈希表在rehashidx索引上的所有键值对rehash到ht[1]，当
rehash工作完成之后，程序将rehashidx属性的值增一。
- 随着字典操作的不断执行，最终ht[0]的所有键值对都会被rehash到ht[1]，这时将rehashidx属性值设为-1。




## 1.4 set

### 1.4.1 简介

Redis Set 相当于 Java重的 HashSet，内部键值对是无序且唯一的，内部实现相当于一个特殊的字典，字典的value都是NULL。

set 结构可以用来存储活动中奖的用户 ID，因为有去重功能，可以保证同一个用户不会中奖两次。

### 1.4.2 使用

```shell
sadd sk v1
sadd sk v2
sadd sk v3 v4
smembers sk
> v3 v1 v4 v2
sismember sk v3
> (integer) 1
scard sk  # 长度
> (integer) 4
spop sk   # 弹出一个
> v3
```

### 1.4.3 底层结构
同Hash

## 1.5 zset

### 1.5.1 简介
ZSet 一方面保证内部value的唯一性，另一方面它可以给每个value赋予一个score，代表这个value的排序权重，内部实现是跳表的数据结构。

ZSet 可以用来存粉丝列表，value 值是粉丝的用户 ID，score 是关注时间。我们可以对粉丝列表按关注时间进行排序。
ZSet 还可以用来存储学生的成绩，value 值是学生的 ID，score 是他的考试成绩。我们可以对成绩按分数进行排序就可以得到他的名次。

### 1.5.2 使用
```shell
zadd zk 9.0 v1
zadd zk 8.9 v2
zadd zk 8.6 v3
zrange zk 0 -1 # score，参数区间为排名范围
> v3
> v2
> v1
zrevrange zk 0 -1 # 逆序
> v1
> v2
> v3
zcard zk #总数
> (integer) 3
zscore zk v1 #获取score
> 8.9000000000000004 # double进行存储，所以存在精度问题。
zrank zk v1 # 排名
> (integer) 1
zrangebyscore zk 0 8.91 # 根据分值区间遍历
zrangebyscore zk -inf 8.91 withscores #根据分值区间 (-∞, 8.91] 遍历 zset，同时返回分值。inf 代表 infinite，无穷大的意思。
zrem zk v1 # 删除
zrange zk 0 -1 #遍历
```

### 1.5.3 底层结构
同Hash




