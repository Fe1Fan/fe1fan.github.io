---
title: "Java基础知识1-容器"
date: 2020-10-07T14:23:52+08:00
tags: [java基础]
---

---
<!--more-->
在Java中，容器主要分为一下两类
- Collection
- Map


## Collection

主要实现关系如下：

---
- Collection `Interface`
    - Set `Interface`
        - SortedSet `Interface`
            - TreeSet `Class`
        - HashSet `Class`
        - LinkedHashSet `Class`
        
    - List `Interface`
        - ArrayList `Class`
        - Vector `Class`
        - LinkedList `Class`
    - Queue `Interface`
        - LinkedList `Class`
        - PriorityQueue `Class`
---

### Set

#### TreeSet

基于红黑树，支持有序的操作，例如根据范围查找，但是效率不如HashSet。

#### HashSet

基于哈希表实现，支持快速查找，失去了元素的插入顺序信息。

#### LinkedHashSet

具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

### List

#### ArrayList
