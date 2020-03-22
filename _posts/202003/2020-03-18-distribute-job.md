---
layout: post
title:  "分布式任务调度系统"
date:   2018-07-11 12:00:00

categories: tool
tags: archtecture refactoring tool
author: "XueGang Zhang"
image: /images/logo.png
comments: true
published: true
---

#### 背景介绍

针对分布式跑批任务进行调研，选择合适于当前场景下的任务调度系统，对其进行改造以应用于生产环境。任务调度最直接简单的方式就是利用Lunix自带的任务Crontab，但是在任务执行监控、日志查看等任务管理上都存在很多的问题。现在的开源系统分为两类：以 Quartz 为代表的定时类调度系统和以 DAG 为核心的工作流调度系统。首先看看定时类调度系统，它们的设计核心是定时运行、数据分片和弹性扩容，但是对依赖关系支持的不太友好，更适用于后端业务开发，其代表为 XXL-JOB 、Elastic-Job 。而数据团队最常见的操作是的 ETL （抽取、转换和加载数据），更强调的是任务的依赖关系，所以关注点便是以 DAG 为核心的工作流调度系统了。




文章参考：
[闲聊调度系统 Apache Airflow](https://zhuanlan.zhihu.com/p/100526494)
