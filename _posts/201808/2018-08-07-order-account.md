---
layout: post
title:  "订单台账系统"
date:   2018-07-11 12:00:00

categories: java
tags: shareArticle
author: "张学刚"
---

### **支付系统切换**

背景：支付系统切换为京东金融的聚合支付系统，支付系统的职责，承担用户支付，以及写台账实收与资金帐的职责，相应的7fresh财务系统如何配合做出调整呢？这个地方有个问题，就是父子单情况的时候，由谁来写实收和资金帐的问题，因为支付系统并不关注子单。无法获取子单号，但是资金帐和台账系统需要体现子单。

多个来写的时候，金额校验

##流程修改前##

![台账系统流程交互](https://raw.githubusercontent.com/unionstars/unionstars.github.io/master/assets/images/pictures/2018-08-07-pay-switch/01-01.png)

正向流程

``` bash

@startuml
订单系统 -[#red]> 台账系统 :下单写应收(RPC)
支付系统 -[#red]> 台账系统 :支付成功写实收(RPC)
支付系统 -
@enduml

```

