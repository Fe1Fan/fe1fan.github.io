---
title: "Redis 笔记8：实际应用"
date: 2020-10-07T14:23:52+08:00
tags: [redis]
math: true
draft: true
---
<!--more-->

## 分布式锁

`setnx`

如果不存在，则设置一个Key，

```shell
setnx key value
expire key 6
```



问题：
- 两条命令并非原子操作，假如setnx执行失败，expire 也会继续执行。
- 执行过程中超时
- 超时后删除key命令删除的不是自己的，因为自己的已经超时。
- 可重入性。

2.8中对set进行扩展，

```shell
set key value ex 5 nx
```

超时问题暂时没有好的解决方案，执行过程中检查key剩余时间进行续期？

生成key和删除key的时候生成随机数，删除时判断是否是自己生成的。

在Java中 ThreadLocal 中存储锁，每次检查本线程是否拿到锁，然后+1。

## 异步消息

使用list作为延时队列

rpush/lpush操作入队列，使用lpop 和 rpop来出队列。

队列消费完成空的造成死循环？

- sleep(1) 通知不及时
- 使用阻塞读 blpop/brpop，阻塞读在队列没有数据的时候，会立即进入休眠状态。但是长时间闲置的空连接会被断开。


## 延时队列
使用zset

```shell
zset key value 时间戳
```

取的时候按照时间戳排序取，使用zrem解决并发问题。

## 位图

gitbit/setbit

## 布隆过滤器
使用多个hash算法对key进行hash运算，然后对位数组长度取模运算，获取到多个位置。
把这些位置值都设置成1，当判断是否存在时，取所有位置，是否都为1，如果有一个为0则不存在，
如果都为1，也不一定存在。

应用场景：
- 爬虫重复URL
- NoSQL，过滤不存在的行，如果存在才去查询IO。
- 邮件过滤系统，垃圾过滤。

使用：
```shell
127.0.0.1:6379> bf.add codehole user1
(integer) 1
127.0.0.1:6379> bf.add codehole user2
(integer) 1
127.0.0.1:6379> bf.add codehole user3
(integer) 1
127.0.0.1:6379> bf.exists codehole user1
(integer) 1
127.0.0.1:6379> bf.exists codehole user2
(integer) 1
127.0.0.1:6379> bf.exists codehole user3
(integer) 1
127.0.0.1:6379> bf.exists codehole user4
(integer) 0
127.0.0.1:6379> bf.madd codehole user4 user5 user6
1) (integer) 1
2) (integer) 1
3) (integer) 1
127.0.0.1:6379> bf.mexists codehole user4 user5 user6 user7
1) (integer) 1
2) (integer) 1
3) (integer) 1
4) (integer) 0
```

## 限流
系统要限定用户的某个行为在指定的时间里只能允许发生 N 次

用一个 zset 结构记录用户的行为历史，每一个行为都会作为 zset 中的一个 key 保存下来。同一个用户同一种行为用一个 zset 记录。

### 漏洞限流
- 将每个请求视作"水滴"放入"漏桶"进行存储；
- “漏桶"以固定速率向外"漏"出请求来执行如果"漏桶"空了则停止"漏水”；
- 如果"漏桶"满了则多余的"水滴"会被直接丢弃。

### 令牌桶
- 令牌以固定速率生成；
- 生成的令牌放入令牌桶中存放，如果令牌桶满了则多余的令牌会直接丢弃，当请求到达时，会尝试从令牌桶中取令牌，取到了令牌的请求可以执行；
- 如果桶空了，那么尝试取令牌的请求会被直接丢弃。
