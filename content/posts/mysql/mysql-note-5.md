---
title: "MySQL 笔记5：事务"
date: 2020-10-07T14:23:52+08:00
tags: [mysql]
---
<!--more-->
## 1.0 InnoDB 事务

MySQL 中有一个系统变量 autocommit，默认情况下，如果我们不显式的使用START TRANSACTION或者BEGIN语句开启一个事务，那么每一条语句都算是一个独立的事务，这种特性称之为事务的自动提交。

如果关闭 autocommit， `SET autocommit = OFF;` 这样的话，我们写入的多条语句就算是属于同一个事务了，直到我们显式的写出COMMIT语句来把这个事务提交掉，或者显式的写出ROLLBACK语句来把这个事务回滚掉。.


### 1.1 隔离级别
- 读未提交
  
  可以读取未提交事务已经更改的数据
  
- 读已提交
  
  只能读取到已提交事务更改后的数据，每次执行SQL前刷新视图。
  
- 可重复读
  
  在初始化时生成视图，整个事务过程读取的内容一致，但是提交更新的时候，数据是按照最新视图来。
  
- 串行
  
  。。。


### 1.2 隐式提交
当我们使用START TRANSACTION或者BEGIN语句开启了一个事务，或者把系统变量autocommit的值设置为OFF时，事务就不会进行自动提交，但是如果我们输入了某些语句之后就会悄悄的提交掉，就像我们输入了COMMIT语句了一样，这种因为某些特殊的语句而导致事务提交的情况称为隐式提交，这些会导致事务隐式提交的语句包括：

- 定义或修改数据库对象的数据定义语言（Data definition language，缩写为：DDL）。

  所谓的数据库对象，指的就是数据库、表、视图、存储过程等等这些东西。当我们使用CREATE、ALTER、DROP等语句去修改这些所谓的数据库对象时，就会隐式的提交前边语句所属于的事务。
  
- 隐式使用或修改mysql数据库中的表

  当我们使用ALTER USER、CREATE USER、DROP USER、GRANT、RENAME USER、REVOKE、SET PASSWORD等语句时也会隐式的提交前边语句所属于的事务。
  
- 事务控制或关于锁定的语句

  当我们在一个事务还没提交或者回滚时就又使用START TRANSACTION或者BEGIN语句开启了另一个事务时，会隐式的提交上一个事务，

- 加载数据的语句

  比如我们使用LOAD DATA语句来批量往数据库中导入数据时，也会隐式的提交前边语句所属的事务。

- 关于MySQL复制的一些语句

  使用START SLAVE、STOP SLAVE、RESET SLAVE、CHANGE MASTER TO等语句时也会隐式的提交前边语句所属的事务。

- 其它的一些语句

  使用ANALYZE TABLE、CACHE INDEX、CHECK TABLE、FLUSH、 LOAD INDEX INTO CACHE、OPTIMIZE TABLE、REPAIR TABLE、RESET等语句也会隐式的提交前边语句所属的事务。

### 1.3 保存点
如果你开启了一个事务，并且已经敲了很多语句，忽然发现上一条语句有点问题，你只好使用ROLLBACK语句来让数据库状态恢复到事务执行之前的样子，然后一切从头再来，总有一种一夜回到解放前的感觉。

保存点就是在事务对应的数据库语句中打几个点，我们在调用ROLLBACK语句时可以指定会滚到哪个点，而不是回到最初的原点。

```sql
SAVEPOINT 保存点名称;
```

当我们想回滚到某个保存点时:

```sql
ROLLBACK [WORK] TO [SAVEPOINT] 保存点名称;
```

删除保存点：

```sql
RELEASE SAVEPOINT 保存点名称;
```

//TODO redo
//TODO undo
