---
layout: post
title:  "订单金额计算系统"
date:   2018-07-11 12:00:00
categories: Java
tags: learningnote
author: "张学刚"
---

# 订单台账中心

>A order pay center for order.

## 业务概述

当用户成功购买商品后，需要申请退款或者退货退款，可发生在订单待发货，待收货，或是已完成状态。 退款一般是由售后系统发起，仅退款发生状态：待发货、待收货状态。退货退款发生状态： 待收货或订单完成状态。

### 系统上下文

## 名词解释

### 退款审批状态

```java
UNAPPROVED(1, "未审批"),
PASSED(2, "审批通过"),;
```

### 资产类型
订单中心在订单接单的时候写台账应收

- 售后系统

- 支付系统

支付成功后，写实收

- 虚拟资产系统

- 积分系统

### orderSource 订单主体来源

- 7fresh
- JD
- 到家
- 华润订单

### originalBillId 原始单据编号

- 正向传订单号
- 逆向传服务单号

### orderPlatform 订单平台类型

- APP(1, "APP"),
- POS(2, "POS"),
- AUTO_BUY_CAR(3, "AUTO_BUY_CAR"),
- HTML5(4, "餐饮自助H5"),//餐饮 Html5 App
- M(5, "7FRESH公共H5"),//7FRESH公共H5

### businessType 业务类型

- ORDER(1000, "订单"),
- AFTER_SALE(1001, "售后服务单"),
- ADJUSTMENT(1002, "调整单"),
- COMPENSATE(1003, "赔付单"),
- CANCEL(1004, "售前取消订单")

### reItemType 应收类型

- 1000:商品金额
- 2000:单品直降
- 2001:单品折扣
- 2002:单品赠品
- 2400:满减
- 2401:满折
- 2402:满赠
- 2600:打折码
- 3000:基础运费

## PayAssetTypeEnum  实收表记录资产类型

- MONEY_PAY(1001, "货币", 1, 1, 1001,null),
- CASH_COUPON(2001, "代金券", 2, 1,UserCouponTypeEnum.CASH_COUPON.getCode(),null),
- SubCouponTypeEnum.SIMPLE.getCode()),
- FREIGHT_COUPON(2003, "运费券", 2, 1,UserCouponTypeEnum.EXPRESS_COUPON.getCode(),null),

### 对账状态

- 100, "待对账"
- 101, "对账完成" //终态
- 104，“对账失败” //终态
- 105，“已核销” //终态
- 106, "重复支付" //终态
- 108，“多退款” //终态

## 业务流程

### 系统交互流程图

![系统交互流程图](https://raw.githubusercontent.com/unionstars/unionstars.github.io/master/assets/images/pictures/2018-08-07-order-account-business/01-01.png)
