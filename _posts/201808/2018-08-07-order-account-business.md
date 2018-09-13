---
layout: post
title:  "订单台账业务流程"
date:   2018-07-11 12:00:00

categories: Java
tags: learningnote
author: "张学刚"
---

# 订单台账中心

>A Account center for order.

## 业务概述

## 名词解释

### 数据来源

- 订单中心
- 售后系统
- 支付系统
- 虚拟资产系统
- 积分系统

### orderSource 订单主体来源

- 7fresh
- JD

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