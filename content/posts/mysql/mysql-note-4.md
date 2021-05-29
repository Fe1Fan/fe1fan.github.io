---
title: "MySQL 笔记4：锁"
date: 2020-10-07T14:23:52+08:00
tags: [mysql]
draft: true
---
<!--more-->
## 1.1 锁类型
- 全局锁
- 表级锁
- 行级锁


## 2.0 全局锁

对整个数据库加锁，MySQL本身提供加锁方法 `FLUSH TABLS WITH READ LOCK`，之后其它线程的增删改、创建表、修改表、更新类事务语句都会被阻塞。


## 3.0 表级锁
