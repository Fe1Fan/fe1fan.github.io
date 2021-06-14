---
title: "Redis 笔记4：持久化"
date: 2020-10-07T14:23:52+08:00
tags: [redis]
math: true
draft: true
---
<!--more-->
# 快照和追加

## 快照
定时对当前内存进行快照，保存合同部。

## 追加
记录每条操作

```shell
appendfsync always    #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
appendfsync everysec  #每秒钟同步一次，显示地将多个写命令同步到硬盘
appendfsync no        #让操作系统决定何时进行同步
```

一般使用 每秒同步。
