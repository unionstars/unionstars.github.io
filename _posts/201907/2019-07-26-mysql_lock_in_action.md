---
layout: post
title:  "MySQL 死锁问题分析"
date:   2018-07-11 12:00:00

categories: mysql
tags: deadlock mysql database
author: "XueGang Zhang"
image: /images/logo.png
comments: true
published: true
---

#### 背景介绍
我们在进行互联网应用开发的时候，高并发场景下，很容易遇到死锁的问题，我们从jdbc抛出的死锁异常中，很难看出死锁发生的具体原因，jdbc只是给了一个死锁异常，
但是并没有抛出导致死锁的原因，这是因为mysql本身，发生死锁的时候就没有抛出更多的错误信息。MySQL/InnoDB的加锁分析，对应用开发来说也是比较复杂的，因为
锁这一块的复杂性，很多关于数据库锁的文章，并没有实际的验证，而是似是而非猜测性的，有一些误导。这里是对一个insert和update同一索引数据导致的死锁案例的
分析过程，主要是描述一个解决问题的思路，供大家参考。


#### 发现问题
X同学在生产上发现了如下的死锁异常，想让我一起排查下，登陆服务器后，看到了类似如下的异常信息：

```sql
在分库:[ worker_10~~>jdbc:mysql://hostname:3306/dbname?user=user ],
执行SQL:[ UPDATE tablename SET col1 = col1 + 20,  modified_date = NOW() WHERE order_id = 'xxx' ], 
发生异常:Deadlock found when trying to get lock; try restarting transaction; 
```


#### 排查问题

我们从日志上看，看不出死锁发生的具体原因，我翻看了发生异常堆栈的代码，类似于我之前做的一个订单台账的死锁问题，因为上次排查的时候，就参阅了不少资料，
有些资料也有一定的误导性，借此我把这次死锁原因复现一下，通过实际的操作来分析一下死锁的原因，并分析一下，如何解决此场景的死锁问题。

我们可以通过show engine innodb status，来查看死锁的日志，看看是否可以看出，是执行哪几个SQL引起的，一下是我找dba拿出的日志，我做了一下脱敏。

```sql
2019-07-18 10:03:03 7f16ff826700
*** (1) TRANSACTION:
TRANSACTION 46497170213, ACTIVE 0 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1184, 2 row lock(s)
MySQL thread id 118471353, OS thread handle 0x7f1c2fe77700, query id 146140919609 10.240.24.25 dbname updating
UPDATE tablename SET col1 = col1 + 20,  modified_date = NOW() WHERE order_id = 'xxx' 
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2632 page no 5065 n bits 440 index `uniq_table_name_col1` of table `tablename` trx id 46497170213 lock_mode X locks rec but not gap waiting
Record lock, heap no 374
*** (2) TRANSACTION:
TRANSACTION 46497170214, ACTIVE 0 sec starting index read, thread declared inside InnoDB 5000
mysql tables in use 1, locked 1
3 lock struct(s), heap size 1184, 2 row lock(s)
MySQL thread id 118465375, OS thread handle 0x7f16ff826700, query id 146140919617 172.25.213.222 dbname updating
UPDATE tablename SET col1 = col1 + 30,  modified_date = NOW() WHERE order_id = 'xxx' 
*** (2) HOLDS THE LOCK(S):
    RECORD LOCKS space id 2632 page no 5065 n bits 440 index `uniq_table_name_col1` of table `tablename` trx id 46497170214 lock mode S rec but not gap
Record lock, heap no 374
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2632 page no 5065 n bits 440 index `uniq_table_name_col1` of table `tablename` trx id 46497170214 lock_mode X locks rec but not gap waiting
Record lock, heap no 374
*** WE ROLL BACK TRANSACTION (2)
```

#### 解读死锁日志
![死锁日志解读](/assets/images/pictures/2019-07-26-mysql_local_in_action/20190725100725364_422396275.png?style=centerme)

我们先来看读一下这个死锁日志，主要关注上边红色的内容，读完之后，是不是发现死锁的日志不全？TRANSACTION2中持有的S锁（lock mode S），这里我们看不出是谁添加的。
这也是死锁分析难的一个原因， 这个时候我们需要回到业务上,了解整个事务的逻辑。虽然我们没有获取到解决死锁的全部信息，不过它仍然给了我们很有用的信息，
我们定位到了是哪两条SQL引起的死锁，并在事务2中看到了一个S锁，这个地方可以看一下执行SQL的程序，尤其关注一下S锁是怎么添加的。经过排查，我发现程序中有行代码是这么写的：

```java
try {
    xxxService.insert(xx);
} catch(DuplicateKeyException duplicateKeyException) {
    xxxService.updateXxByIdx(xx);
}
```

#### 已有的知识

又是insert问题导致的死锁？

我们先看一下mysql官方给出的insert导致死锁的案例，然后继续分析一下，这个地方可能导致的死锁问题，之前解决过类似的场景，可以参考这个 [inert死锁](https://dev.mysql.com/doc/refman/5.6/en/innodb-locks-set.html)。

> If a duplicate-key error occurs, a shared lock on the duplicate index record is set. This use of a shared lock can result in deadlock should there be multiple sessions trying to insert the same row if another session already has an exclusive lock. Suppose that an InnoDB table t1 has the following structure:

大概解释一下这句话，当发生duplicate-key错误的时候，会对索引记录加S锁。当有一个session持有了X锁，然后又有多个session同时去插入相同行的时候可能会导致死锁， 我们来看一下这个案例：

```sql
-- table structure
CREATE TABLE t1 (i INT, PRIMARY KEY (i)) ENGINE = InnoDB;

-- Session 1:
START TRANSACTION;
INSERT INTO t1 VALUES(1);

-- Session 2:
START TRANSACTION;
INSERT INTO t1 VALUES(1);

-- Session 3:
START TRANSACTION;
INSERT INTO t1 VALUES(1);

-- Session 1:
ROLLBACK;
```

> The first operation by session 1 acquires an exclusive lock for the row. The operations by sessions 2 and 3 both result in a duplicate-key error and they both request a shared lock for the row. When session 1 rolls back, it releases its exclusive lock on the row and the queued shared lock requests for sessions 2 and 3 are granted. At this point, sessions 2 and 3 deadlock: Neither can acquire an exclusive lock for the row because of the shared lock held by the other.

解释一下这句话： session1拿到了X锁，session2和session3发生duplicate-key错的时候，同时去请求S锁。当session1回滚，它释放X锁，此时session2和session3同时获得S锁，并同时去请求X锁。引起了死锁。我们看一下这个持有和竞争的关系：

![](/assets/images/pictures/2019-07-26-mysql_local_in_action/20190725123235910_1658579667.png?style=centerme)

这个地方是核心要理解锁的一个互斥和共存的关系，我们看一下这个关系：

|     |   S   |  X  |
| --- | ----- | --- |
| S   | 不冲突 | 冲突 |
| X   | 冲突   | 冲突 |

因为X和S锁是互斥的，session2想要X锁，必须等待session3的S锁释放， session3想要获得X锁也要session2释放S锁，这个时候构成了环路等待，引起了死锁。

#### 回到我们案例

很明显我们的案例，不是单纯的insert引起的，不过思考一下，这个地方存在同样的问题，session2和session3如果发生duplicate-key错的时候，同时去请求S锁。然后去执行update这条数据的时候，update需要
加X锁，当他们请求X锁的时候，需要对方释放S锁，这个时候构成了环路等待，引起了死锁。我们来模拟一下这个场景, 这个地方，我们假设数据已经存在了，上边语句的表名，我脱敏了主要是因为是线上信息，接下来，
我就直接创建一份测试数据去模拟了：

```sql
-- table structure
CREATE TABLE `tablename` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `col1` varchar(32)DEFAULT NULL COMMENT '订单号',
  `out_money` bigint(20) DEFAULT '0' COMMENT '转出金额',
  `in_money` bigint(20) DEFAULT '0' COMMENT '转入金额',
  `created_date` datetime NOT NULL COMMENT '创建时间',
  `modified_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
  `mer_date` varchar(8) COLLATE utf8_bin DEFAULT NULL COMMENT '时间切分键',
  `in_settle_money` bigint(20) DEFAULT '0' COMMENT '结算转入金额',
  `out_settle_money` bigint(20) DEFAULT '0' COMMENT '结算转出金额',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uniq_order_account_3413` (`order_id`)
) ENGINE=InnoDB AUTO_INCREMENT=220016 DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='订单金额表';

-- transaction 1
begin;
insert  into order_account_info_3413 values(200014,2000,0,0,now(),now(),20180101,0,0);

-- transaction 2
begin;
insert  into order_account_info_3413 values(200014,2000,0,0,now(),now(),20180101,0,0);

-- transaction 1
update order_account_info_3413 set in_money= in_money + 1 where order_id = 2000;

-- transaction 2
update order_account_info_3413 set in_money= in_money - 1 where order_id = 2000;

-- transaction 2
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction~~~~
```
#### 问题真的只是这样么？

回来再看一下这个代码：

```java
if (xxxService.getXxByIdx(XX) != null) {
    try {
        xxxService.insert(xx);
    } catch(DuplicateKeyException duplicateKeyException) {
        xxxService.updateXxByIdx(xx);
    }
} else {
        xxxService.updateXxByIdx(xx);
}
```

假设程序的执行状态是这样呢？ 会不会存在问题？ 线程1经过1和2到达3，线程2经过1到达2.

![](/assets/images/pictures/2019-07-26-mysql_local_in_action/20190725145431560_396427721.png?style=centerme)

我们模拟一下这个场景：

```sql
-- transaction 1
begin;
insert  into order_account_info_3413 values(200014,2000,0,0,now(),now(),20180101,0,0);

-- transaction 2
begin;
update order_account_info_3413 set in_money= in_money + 1 where order_id = 2000;

-- transaction 1
update order_account_info_3413 set in_money= in_money - 1 where order_id = 2000;


-- transaction 2
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction~~~~
```

## why？

晕了，分析一下这个过程吧

| time |                                                              session 1                                                              |                                                  session2                                                  |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| 1    | begin;                                                                                                                              |                                                                                                            |
| 2    | insert  into order_account_info_3413 values(200014,2000,0,0,now(),now(),20180101,0,0);唯一索引冲突，对唯一索引加S锁                   |                                                                                                            |
| 3    |                                                                                                                                     | begin;                                                                                                     |
| 4    |                                                                                                                                     | update order_account_info_3413 set in_money= in_money + 1 where order_id = 2000;  加X锁，等待session1的S锁 |
| 5    | update order_account_info_3413 set in_money= in_money - 1 where order_id = 2000;这个地方应该做了死锁检查，认为死锁，然后把session2回滚了 |                                                                                                            |
| 6    |                                                                                                                                     | ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction~~~~               |

这个地方不太明白，死锁检查的时候，为什么判定是死锁了，猜一下吧，因为session2先进入了等待X锁的队列，但是他需要session1的S锁，session1需要拿X锁，但是因为session2线进入了等待X锁的队列，所以它需要等待session2 拿到X锁后释放S锁。好像可以说通！


#### 验证一下

如果还是不太理解这个地方可以debug一下mysql的执行，核心就是死锁检测的地方，他是怎样认为存在死锁，并对session2进行回滚的，我们可以下载mysql的源码，进行编译和调试，核心找到Innodb中检测与处理死锁的代码入口：
我们发现，死锁检测函数是storage/innobase/lock/lock0lock.c的lock_deadlock_occurs函数，这个函数调用了lock_deadlock_recursive函数迭代地检查死锁：

```c
static
ulint
lock_deadlock_recursive(
/*====================*/
	trx_t*	start,		/*!< in: recursion starting point */
	trx_t*	trx,		/*!< in: a transaction waiting for a lock */
	lock_t*	wait_lock,	/*!< in:  lock that is waiting to be granted */
	ulint*	cost,		/*!< in/out: number of calculation steps thus
				far: if this exceeds LOCK_MAX_N_STEPS_...
				we return LOCK_EXCEED_MAX_DEPTH */
	ulint	depth);		/*!< in: recursion depth: if this exceeds
				LOCK_MAX_DEPTH_IN_DEADLOCK_CHECK, we
				return LOCK_EXCEED_MAX_DEPTH */

```
lock_deadlock_recursive是迭代的主函数。 start为初始事务, wait_lock为待判断锁， cost和depth为累积的消耗和迭代深度。


里边有一个非常重要的死锁检测逻辑，当事务尝试获取（请求）加一个锁，并且需要等待时，innodb会开始进行死锁检测。
```c
	if (lock_has_to_wait(wait_lock, lock)) {
			ibool	too_far
				= depth > LOCK_MAX_DEPTH_IN_DEADLOCK_CHECK
				|| *cost > LOCK_MAX_N_STEPS_IN_DEADLOCK_CHECK;

			lock_trx = lock->trx;

			if (lock_trx == start) {

				/* We came back to the recursion starting
				point: a deadlock detected; or we have
				searched the waits-for graph too long */
			}
		}

```
我们发现当函数第二次递归的时候，lock_trx 等于 start，成环死锁产生。要不再精进一下？我们分析一下，这两次递归都干了什么？这个问题我和[挖坑的张师傅](https://mp.weixin.qq.com/s/9GhchWWsUqNvN6-bWL-olA)
一起探讨过，他把debug的思路写了下来，是个不错的参考。

#### 怎么解决？

- 第一种方案：`insert  into order_account_info_3413 values(200014,2000,0,0,now(),now(),20180101,0,0) on duplicate key update in_money= in_money + 1;`？
- 第二种方案：分布式锁?
- 第三种方案：insert 发生duplicate-key不执行更新了直接抛异常重新走一下？

第一种解决方案，[咖啡拿铁](https://juejin.im/user/57e4a4e80e3dd9005809b6fb)同学说，依然存在死锁问题，这个地方我在RC和RR的隔离级别下没有重现这个异常。
暂且认为这种方式是可用的，这种方式的有优点mysql就提供了这种保护机制，不用通过程序去保护，缺点就是把这个责任交给了mysql，一般来说，数据库资源更宝贵一些，而且这种写法也有人
反馈有一定的问题，尤其是存在多个唯一索引的时候。所以在使用的时候，需要一定的压测，或者对mysql更系统的学习。

第二种方案，分布式锁，是一个比较简单的方案，把并行化修改为串行化进入，效率也不会太差，一般在互联网应用中，其实还是比较常见的方式，缺点就是引入了新的组件，这个地方还有一个缺
点就是，如果这是一个对外的接口，这个地方还需要考虑接口的幂等以及可用率，这里超出了本文的讨论范围。

第三种方案，依赖于重试策略，这种方式简单，但是如果是对外接口的话，依然要考虑接口可用率的问题。

技术上没有绝对的最佳实践，最佳实践都是基于业务场景去考虑，这里可以基于自己的业务场景去选择和考虑更好的解决方案。

#### 总结

以上的实践都在RC事务隔离级别下完成，互联网的解决方案多是碎片化的，作为我们要学会甄别信息的准确性，并进行一定的尝试和实验。尽量对官方的文档进行系统的学习，网络上的文章作为
理解的辅助参考。然后慢慢形成自己的只是体系。


本文中还参考了如下内容：
    
[mysql官方手册](https://dev.mysql.com/doc/refman/5.6/en/innodb-deadlocks.html)

[何登成的技术博客](http://hedengcheng.com/?p=771)
