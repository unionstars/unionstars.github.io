---
layout: post
title:  "深度解密京东订单履约之旅"
date:   2018-07-11 12:00:00

categories: Java
tags: learningnote
author: "张学刚"
---

`此文件描述核心业务`

# 订单履约中心

>A opc center for order.

## 业务概述

在电子商务企业中，企业通过优质商品、促销核心追求的就是能与消费者进行交易，而订单可以认为是一次交易生命周期的过程，交易开始生成订单，结束的时候完成订单。交易的核心要素也就是订单上的商品信息、发票（增值税发票，还是普通发票）、运费、时效、预约、优惠等等相关内容，都是订单关心的内容，而这些内容如何被正确的跟踪与执行，是OPC系统核心要解决的问题。

本文的内容，是通过京东现有资料整理而成，但是并不代表京东的立场，本书的内容很多也是个人的思考和个人的观点，并非学术性的结论，对于文章的观点，欢迎大家质疑和思考，本书的目的也是想引发大家的思考。

## 系统介绍

### 何为订单履约

![订单履约](https://raw.githubusercontent.com/unionstars/unionstars.github.io/master/assets/images/pictures/2018-09-07-order-opc-business/0001.jpg)
