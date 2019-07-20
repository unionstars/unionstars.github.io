---
layout: post
title:  "mysql 死锁问题分析"
date:   2018-07-11 12:00:00

categories: mysql
tags: deadlock mysql database
author: "张学刚"
---

##### 背景

MySQL/InnoDB的加锁分析，一直是一个比较困难的话题。我在工作过程中，经常会有同事咨询这方面的问题。
同时，微博上也经常会收到MySQL锁相关的私信，让我帮助解决一些死锁的问题。本文，准备就MySQL/InnoDB的加锁问题，展开较为深入的分析与讨论，
主要是介绍一种思路，运用此思路，拿到任何一条SQL语句，都能完整的分析出这条语句会加什么锁？会有什么样的使用风险？甚至是分析线上的一个死锁场景，了解死锁产生的原因。
