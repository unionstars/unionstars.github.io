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

台帐就是明细纪录表。台帐，不属于会计核算中的帐簿系统，不是会计核算时所记的帐簿，它是企业为了加强某方面的管理、更加详细地了解某方面的信息而设置的一种辅助帐簿，没有固定的格式，没有固定的帐页，企业可根据实际需要自行设计，尽量详细，以全面反映某方面的信息，不必按凭证号记帐，但能反映出记帐号更好。具体在仓库管理中，台账详细记录了什么时间，什么仓库，什么货架由谁入库了什么货物，共多少数量。同样，出库也要对应出库台账。再说的明白一点，台账就是流水账。

订单台账系统正向接收订单系统的应收数据，支付系统的实收数据，逆向接收售后系统应收数据，退款系统的实收数据，应收和实收，都会触发对账任务，对账完成后，订单中间件(xstore-order-middleware)接收对账结果，并下发给OPC(订单生产中心)控制订单生产。

## 名词解释

### 数据来源

- 订单中心

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


