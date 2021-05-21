---
title: "Redis 笔记1：基础"
date: 2020-10-07T14:23:52+08:00
tags: [redis]
math: true
---
<!--more--> 
# 基础数据结构

- string
- list
- hash
- set
- zset

## string

### 简介
string是Redis中最常用的数据结构，其长度是可变的，采用的内存分配策略是
预分配冗余空间的方式来减少内存的频繁分配，当字符串长度小于 1M 时，扩容都是加倍现有的空间，如果超过 1M，扩容时一次只会多扩 1M 的空间。
需要注意的是字符串最大长度为 512M。

### 使用
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

## list

### 简介
Redis中的list是双向链表而不是数组，这意味着插入和删除非常快，但是索引很慢。当列表弹出了最后一个元素之后，该数据结构自动被删除，内存被回收。
因为底层是连续的内存块，所以速度很块。

### 使用
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

## hash

### 简介
Redis Hash 使用数组 + 链表二维结构，如果hash碰撞，则放到对应hash的链表中。
Redis的值只能是字符串，

### 使用

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
