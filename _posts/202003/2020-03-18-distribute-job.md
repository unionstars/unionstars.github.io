---
layout: post
title:  "分布式任务调度系统实现原理与选型"
date:   2018-07-11 12:00:00

categories: tool
tags: archtecture refactoring tool
author: "XueGang Zhang"
image: /images/logo.png
comments: true
published: true
---

### 背景介绍

针对分布式跑批任务进行调研，选择合适于当前场景下的任务调度系统，对其进行改造以应用于生产环境。任务调度最直接简单的方式就是利用Lunix自带的任务Crontab，但是在任务执行监控、日志查看等任务管理上都存在很多的问题。现在的开源系统分为两类：以 Quartz 为代表的定时类调度系统和以 DAG 为核心的工作流调度系统。首先看看定时类调度系统，它们的设计核心是定时运行、数据分片和弹性扩容，但是对依赖关系支持的不太友好，更适用于后端业务开发，其代表为 XXL-JOB 、Elastic-Job 。而数据团队最常见的操作是的 ETL （抽取、转换和加载数据），更强调的是任务的依赖关系，所以关注点便是以 DAG 为核心的工作流调度系统了。

### 任务调度系统核心功能

- 任务分片：对于一个大型任务，需要支持分片并行执行。
- 高可用: 调度系统自身必须保证高可用。
- 任务编排： 任务与任务之间可以进行次序的编排。

### 主流分布式调度技术比较
|     |   elastic-job   |  XXL-JOB  |easyJob|
| --- | ----- | --- |--- |
| 任务分片   | √ | √ |√|
| 高可用   | 基于zookeper   | 基于数据库 | 基于数据库，不支持失效转移 |
| 任务编排   | × |通过子任务依赖来实现  | 完善的DAG功能 |

### 如何实现失效转移

#### elasticJob 如何实现失效转移

##### HA

当某一个任务实例节点宕机（离开与zookeeper的连接），会触发elastic-job主节点的重新分片逻辑。elastic-job启动任务节点以后生成的zookeeper中的instance节点是一个临时节点EPHEMERAL。为什么要用EPHEMERAL节点，就是为了能在任务实例出现问题与zookeeper断开以后，能触发zookeeper的节点移除的事件，从而重新调整分区或者运行节点。既然是EPHEMERAL节点，就可以在zookeeper中配置sessionTimeoutMs参数。在使用spring的elastic-job配置中在如下地方配置：

![关系图](/assets/images/pictures/2020-03-18-distribute-job/01.png?style=centerme)

如果在sessionTimeoutMs的时间段之内触发任务，则异常分片的任务会丢失。举个例子：假如sessionTimeoutMs被设置成1分钟，而本身的任务是30秒执行一次，有三个任务实例在三台机器各自执行分片1,2,3。当分片3所在的机器出现问题，和zk断开了，那么zk节点失效至少要到1分钟以后。期间30秒执行一次的任务分片3，至少会少执行一次。1分钟过后，zk节点失效，触发ListenServersChangedJobListener类的dataChanged方法，在这里方法中判断instance节点变化，然后通过方法shardingService.setReshardingFlag设置重新分片标志位，下次执行任务的时候，leader节点重新分配分片，分片3就会转移到其他好的机器上。


##### 失效转移

elastic-job的任务配置有个failover，如果开启设置为true的时候，会启动真正的失效转移：，elastic-job的任务又两个配置failover（默认值为false）和monitorExecution（默认值是true）。只有对monitorExecution为true的情况下才可以开启失效转移。
![关系图](/assets/images/pictures/2020-03-18-distribute-job/02.png?style=centerme)

所谓失效转移，就是在执行任务的过程中遇见异常的情况，这个分片任务可以在其他节点再次执行。这个和上面的HA不同，对于HA，上面如果任务终止，那么不会在其他任务实例上再次重新执行。

Job的失效转移监听来源于FailoverListenerManager中JobCrashedJobListener的dataChanged方法。FailoverListenerManager监听的是zk的instance节点删除事件。如果任务配置了failover等于true，其中某个instance与zk失去联系或被删除，并且失效的节点又不是本身，就会触发失效转移逻辑。

首先，在某个任务实例失效时，elastic-job会在leader节点下面创建failover节点以及items节点。items节点下会有失效任务实例的原本应该做的分片号。比如，失效的任务实例原来负责分片1和2。那么items节点下就会有名字叫1的子节点，就代表分片1需要转移到其他节点上去运行。如下图：
![关系图](/assets/images/pictures/2020-03-18-distribute-job/03.png?style=centerme)

然后，由于每个存活着的任务实例都会收到zk节点丢失的事件，哪个分片失效也已经在leader节点的failover子节点下。所以这些或者的任务实例就会争抢这个分片任务来执行。为了保证不重复执行，elastic-job使用了curator的LeaderLatch类来进行选举执行。在获得执行权后，就会在sharding节点的分片上添加failover节点，并写上任务实例，表示这个故障任务迁移到某一个任务实例上去完成。如下图中的sharding节点上的分片1：
![关系图](/assets/images/pictures/2020-03-18-distribute-job/04.png?style=centerme)
执行完成后，会把相应的节点和数据删除，避免下一次重复执行。

#### xxl-JOB 如何实现失效转移

每个任务执行的后，记录执行状态，当服务器宕机时，当前务执行失败的时候，记录失败日志，然后由后台的一个守护线程，搜索执行失败的任务进行重新触发。当触发的任务的时候，会进行多种路由策略的选择，如果客户端选择了故障转移策略。则管理端会将之前保存的addressList与服务端进行一个心跳的检测，检测机器是否有crash的，如果找到存活的就执行调度。如果失联就记录日志。等待下次重试。

#### 如何实现任务分片


#### easyJob 如何实现失效转移

easyJob本身没有失效转换的功能，如果




#### 如何实现数据分片，水平扩容











文章参考：
[闲聊调度系统 Apache Airflow](https://zhuanlan.zhihu.com/p/100526494)
[elasticjob源码分析](https://blog.csdn.net/qq924862077/article/details/83036626)
[elasticjo失效转移](https://www.cnblogs.com/haoxinyue/p/7068115.html)
[xxl-job如何进行稳定性保障](https://blog.csdn.net/royal_lr/article/details/100113760)
