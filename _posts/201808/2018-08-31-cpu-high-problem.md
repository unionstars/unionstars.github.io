---
layout: post
title:  "台账数据库CPU飙升问题处理"
date:   2018-07-11 12:00:00

categories: Java
tags: learningnote
author: "张学刚"
---

# 台账数据库CPU飙升问题处理

## 发现问题

晚上18.54开始，数据库接口性能开始告警，TP999达到6秒，TP99达到2秒多，查看数据库，数据库连接数达到2000，CPU占用上升至80%，单表插入超过2S，查看日志：应用日志中有插入防重表失败的问题。

## 定位问题

因为有针对此问题的应急预案，第一时间见进行了恢复，同时保留了现场，后续回顾问题时进行详细定位

## 快速解决问题

快速恢复、保留现场，因为数据库已经超过阈值，并且响应开始变慢，对应用已经造成比较严重的影响，这个时候应该快速的走应急预案，先实施了线下店削峰方案，对数据进行缓冲处理，在没有迅速定位问题的情况下，对现场的运行状态，应用日志，数据库状态进行了保存，方便后续对问题的回顾排查

问题解决后，切流开关切回到非应急状态

## 回顾问题

### 发现大量慢SQL,SQL的执行时长达到12S,严重影响数据库性能

![大量慢SQL](https://raw.githubusercontent.com/unionstars/unionstars.github.io/master/assets/images/pictures/2018-08-31-problem-analy/01-01.jpg)

排查后发现，慢SQL的原因为新上线了，导出实收的功能，关联查询实收和任务表数据，第一次会生成多月的账单明细，每页数据返回都有性能问题

### 数据库事务时长，超过JSF客户端超时时间，如果JSF设置重试次数，会引起循环调用

```log
2018-08-30 18:54:39.242 [JSF-BZ-22000-17-T-20] INFO com.jd.xstore.order.account.spi.impl.OrderAccountServiceProviderImpl[72] - OrderAccountServiceProviderImpl.recordNetReceipts写实收,recordNetReceipts_orderId=860003001604,originalBillId=860003001604,businessType=1000,request={"requestData":[{"accountSn":"18083010610004911425185437","balance":81.80,"bankConfirmTime":1535626478000,"businessType":1000,"code":"0","direction":1,"orderId":"860003001604","orderPlatform":2,"orderSource":1,"orderTime":1535626478000,"originalBillId":"860003001604","payAssetId":0,"payAssetName":"货币","payAssetType":1001,"payChannel":6,"payId":"180830210004911424185437","storeId":"131231","storeName":"","success":true,"userPin":"jdpos"}],"sourceType":10002}
```

```log
018-08-30 18:54:42.242 [JSF-BZ-22000-17-T-20] INFO com.jd.xstore.order.account.spi.impl.OrderAccountServiceProviderImpl[72] - OrderAccountServiceProviderImpl.recordNetReceipts写实收,recordNetReceipts_orderId=860003001604,originalBillId=860003001604,businessType=1000,request={"requestData":[{"accountSn":"18083010610004911425185437","balance":81.80,"bankConfirmTime":1535626478000,"businessType":1000,"code":"0","direction":1,"orderId":"860003001604","orderPlatform":2,"orderSource":1,"orderTime":1535626478000,"originalBillId":"860003001604","payAssetId":0,"payAssetName":"货币","payAssetType":1001,"payChannel":6,"payId":"180830210004911424185437","storeId":"131231","storeName":"","success":true,"userPin":"jdpos"}],"sourceType":10002}
2018-08-30 18:54:42.404 [JSF-BZ-22000-17-T-20] INFO com.xstore.order.account.service.assist.impl.RecordNetReceiptsServiceImpl[55] - recordNetReceipts start, orderId:860003001604,originalId:860003001604,accountSn:18083010610004911425185437
2018-08-30 18:54:42.422 [JSF-BZ-22000-17-T-20] ERROR com.xstore.order.account.manager.impl.OrderAccountAvoidRepetitionManagerImpl[52] - 插入防重数据失败,orderId=860003001604,param={"accountType":20,"businessType":1000,"businessUUID":"665de57d124c003682c30e290b9dbb63","created":1535626482404,"id":5438639,"modified":1535626482404,"orderId":"860003001604","originalBillId":"860003001604","payChannel":6,"payId":"180830210004911424185437","sourceType":10002}
```

通过上边可以看到，同一个线程，在3秒进行重试调用，排查后发现，支付系统调用JSF接口设置了超时次数 同一线程超时会执行两次

```xml
<jsf:consumer id="orderAccountServiceSaaSProvider"
              interface="com.jd.xstore.order.account.spi.service.OrderAccountServiceProvider"
              protocol="jsf"
              alias="${com.jd.xstore.order.account.spi.service.OrderAccountServiceProvider.group}"
              timeout="3000"
              serialization="hessian"
              retries="2">
</jsf:consumer>
```

### 并发导致的锁等待及死锁问题

按照2的情况(数据库事务时长，超过JSF客户端超时时间，引起重复调用),这个地方还极有可能引起死锁，原因如下：

INSERT sets an exclusive lock on the inserted row. This lock is an index-record lock, not a next-key lock (that is, there is no gap lock) and does not prevent other sessions from inserting into the gap before the inserted row.
 
 insert 会对插入成功的行加上排他锁。这个锁是索引记录锁，不是next-key lock（更不是gap lock），不会阻止其他并发事务往这条记录之前插入记录。

Prior to inserting the row, a type of gap lock called an insert intention gap lock is set. This lock signals the intent to insert in such a way that multiple transactions inserting into the same index gap need not wait for each other if they are not inserting at the same position within the gap. Suppose that there are index records with values of 4 and 7. Separate transactions that attempt to insert values of 5 and 6 each lock the gap between 4 and 7 with insert intention locks prior to obtaining the exclusive lock on the inserted row, but do not block each other because the rows are nonconflicting.

在插入之前，设置了一个叫做insert intention gap lock（插入意向间隙锁）的间隙锁类型。此锁表示插入这样的意图：即插入到同一索引间隙中的多个事务如果不在间隙内的同一位置插入，则不需要彼此等待。（个人理解，insert intention gap lock 不等于其他gap lock。不会阻塞）。假设有索引记录的值为4和7。当拥有4和7的这个insert intention locks在获得x lock之前，两个试图插入5和6的事务不会相互阻塞对方，锁不冲突。

If a duplicate-key error occurs, a shared lock on the duplicate index record is set. This use of a shared lock can result in deadlock should there be multiple sessions trying to insert the same row if another session already has an exclusive lock. This can occur if another session deletes the row. Suppose that an InnoDB table t1 has the following structure:

CREATE TABLE t1 (i INT, PRIMARY KEY (i)) ENGINE = InnoDB;
Now suppose that three sessions perform the following operations in order:

Session 1:
```sql
START TRANSACTION;
INSERT INTO t1 VALUES(1);
```

Session 2:
```sql
START TRANSACTION;
INSERT INTO t1 VALUES(1);
```

Session 3:
```sql
START TRANSACTION;
INSERT INTO t1 VALUES(1);
```

Session 1:

```sql
ROLLBACK;
```

The first operation by session 1 acquires an exclusive lock for the row. The operations by sessions 2 and 3 both result in a duplicate-key error and they both request a shared lock for the row. When session 1 rolls back, it releases its exclusive lock on the row and the queued shared lock requests for sessions 2 and 3 are granted. At this point, sessions 2 and 3 deadlock: Neither can acquire an exclusive lock for the row because of the shared lock held by the other.

这个时候产生的死锁，只能依赖InnoDB自动检测，使一个事务释放锁并回退，另一个事务获得锁，继续完成事务。但在涉及外部锁，或涉及表锁的情况下，InnoDB并不能完全自动检测到死锁，这需要通过设置锁等待超时参数 innodb_lock_wait_timeout来解决。需要说明的是，这个参数并不是只用来解决死锁问题，在并发访问比较高的情况下，如果大量事务因无法立即获得所需的锁而挂起，会占用大量计算机资源，造成严重性能问题，甚至拖跨数据库。我们通过设置合适的锁等待超时阈值，可以避免这种情况发生。可以看到这个时候，如果事务不能及时提交，客户端不断重试插入的时候，这个地方极有可能会产生死锁，一旦产生，只能依赖于数据库引擎的自动检测机器，和锁等待超时机制。

## 复盘问题，重新梳理模块薄弱点

详细查看了业务当时的调用量，与之前对数据库的压测表明，差距很大，通过上边的分析，极有可能是大查询导致数据库性能下降后，接口性能开始下降，客户端超时又不断重试，导致的连锁问题。

## 避免措施

1. 导出数据等操作做读写分离，切换到从库(避免大查询问题)
2. 并发插入上增加分布式锁，不完全依赖于防重表防重，防止防重表同时操作导致的锁竞争问题
3. 退款失败补偿worker中数据只扫描前两分钟，防止数据被重复扫描
4. 对账任务因为应收实收触发的是一个队长任务，并发执行的时候会有锁竞争问题，因为

参考：

https://dev.mysql.com/doc/refman/5.5/en/innodb-locks-set.html

https://help.aliyun.com/knowledge_detail/41705.html