---
layout: post
title:  "订单台账业务流程"
date:   2018-07-11 12:00:00

categories: Java
tags: learningnote
author: "张学刚"
---

`此文件描述核心业务`

# 订单台账中心

>A Account center for order.

## 业务概述

台帐就是明细纪录表。台帐，不属于会计核算中的帐簿系统，不是会计核算时所记的帐簿，它是企业为了加强某方面的管理、更加详细地了解某方面的信息而设置的一种辅助帐簿，
没有固定的格式，没有固定的帐页，企业可根据实际需要自行设计，尽量详细，以全面反映某方面的信息，不必按凭证号记帐，但能反映出记帐号更好。具体在仓库管理中，台账详细记录了什么时间，
什么仓库，什么货架由谁入库了什么货物，共多少数量。同样，出库也要对应出库台账。再说的明白一点，台账就是流水账。

订单台账系统正向接收订单系统的应收数据，支付系统的实收数据，逆向接收售后系统应收数据，退款系统的实收数据，应收和实收，都会触发对账任务，
对账完成后，订单中间件(xstore-order-middleware)接收对账结果，并下发给OPC(订单生产中心)控制订单生产。

## 名词解释

### SourceTypeEnum 数据来源

    ORDER_CENTER(10000, "订单中心"),
    AFTER_SALES(10001, "售后系统"),
    PAYMENT_CENTER(10002, "支付系统"),
    VIRTUAL_ASSETS_CENTER(10003, "虚拟资产系统"),
    INTEGRAL_CENTER(10004, "积分系统"),
    WELFARE_ASSET_SYSTEM(10005, "福利平台系统"),
    FINANCE_BALANCE_SYSTEM(10006, "金融卡系统"),

### orderSource 订单主体来源

    FRESH(1, "7FRESH"),
    JD(2, "JD "),;

### originalBillId 原始单据编号

    正向传订单号
    逆向传服务单号

### orderPlatform 订单平台类型

    APP(1, "APP"),
    POS(2, "POS"),
    AUTO_BUY_CAR(3, "AUTO_BUY_CAR"),
    HTML5(4, "餐饮自助H5"),//餐饮 Html5 App
    M(5, "7FRESH公共H5"),//7FRESH公共H5

### businessType 业务类型

    ORDER(1000, "订单", 1),
    AFTER_SALE(1001, "售后服务单", -1),
    ADJUSTMENT(1002, "调整单", -1),
    COMPENSATE(1003, "赔付单", -1),
    CANCEL(1004, "售前取消订单", -1),
    REPEAT_PAY(1005, "重复支付退款", -1),//由台账系统发起
    RAPID_DELIVERY_AFTER_SALE(1006, "急速达售后单", -1),

### reItemType 应收类型

    //1开头为商品金额
    PRODUCT_PRICE(1000, "商品金额", 1, 1, 1000),

    //2开头为促销类型
    DIRECT_PROMOTION(2000, "单品直降", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_DEPRECIATE_PRICE.getPromoSubType())),
    SINGLE_DISCOUNT(2001, "单品折扣", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_DISCOUNT_PRICE.getPromoSubType())),
    SINGLE_GITFT(2002, "单品赠品", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_GIFT.getPromoSubType())),
    SINGLE_SPIKE(2003, "单品秒杀", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_FLASH_SALE.getPromoSubType())),

    OVER_SUBTRACT(2400, "满减", 2, -1, Integer.valueOf(PromoSubTypeEnum.SUIT_DEPRECIATE_PRICE.getPromoSubType())),
    OVER_DISCOUNT(2401, "满折", 2, -1, Integer.valueOf(PromoSubTypeEnum.SUIT_DISCOUNT_PRICE.getPromoSubType())),
    OVER_GIFT(2402, "满赠", 2, -1, Integer.valueOf(PromoSubTypeEnum.SUIT_GIFT.getPromoSubType())),
    SUIT_THE_N_PIECE_M_DISCOUNT(2043, "每第N件M折", 2, -1, Integer.valueOf(PromoSubTypeEnum.SUIT_THE_N_PIECE_M_DISCOUNT.getPromoSubType())),

    DISCOUNT_CODE(2600, "打折码", 2, -1, Integer.valueOf(PromoSubTypeEnum.DISCOUNT_CODE.getPromoSubType())),


    ITEM_MEMBER_BENEFITS_DEPRECIATE(105, "单品会员权益直降", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_MEMBER_BENEFITS_DEPRECIATE.getPromoSubType())),
    ITEM_MEMBER_BENEFITS_DISCOUNT(106, "单品会员权益折扣", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_MEMBER_BENEFITS_DISCOUNT.getPromoSubType())),

    GROUP_BUY(150, "单品拼团直降", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_GROUP_BUY_DEPRECIATE.getPromoSubType())),
    CUT_PRICE(160, "单品砍价直降", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_CUT_PRICE_DEPRECIATE.getPromoSubType())),
    SUIT_DEPRECIATE_BULK_BUYING(350,"满额直降大宗采购", 2, -1, Integer.valueOf(PromoSubTypeEnum.SUIT_DEPRECIATE_BULK_BUYING.getPromoSubType())),

    //3开头为运费
    BASIC_FREIGHT(3000, "基础运费", 3, 1, 3000),
    RAPID_DELIVERY_FREIGHT(3001, "急速达运费", 3, 1, 3001),
    OVER_SUBTRACT_FREIGHT(3002, "满减运费", 3, -1, 3002),

    //现金抹零
    CASH_ROUND(4000, "现金抹零", 4, -1, 4000),

    ITEM_FORETASTE_DEPRECIATE(103, "单品试吃直降", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_FORETASTE_DEPRECIATE.getPromoSubType())),
    ITEM_FORETASTE_DISCOUNT(104, "单品试吃折扣", 2, -1, Integer.valueOf(PromoSubTypeEnum.ITEM_FORETASTE_DISCOUNT.getPromoSubType())),

    SUIT_RAISE(304, "满额加价购", 2, -1, Integer.valueOf(PromoSubTypeEnum.SUIT_RAISE.getPromoSubType())),
    SUIT_DEPRECIATE_RAISE(305, "满减加价购", 2, -1, Integer.valueOf(PromoSubTypeEnum.SUIT_DEPRECIATE_RAISE.getPromoSubType())),
    SUIT_N_PIECE_M_DISCOUNT(306, "满N件M折", 2, -1, Integer.valueOf(PromoSubTypeEnum.SUIT_N_PIECE_M_DISCOUNT.getPromoSubType())),
    STACK_DEPRECIATE(500,"组套直降", 2,-1,Integer.valueOf(PromoSubTypeEnum.STACK_DEPRECIATE.getPromoSubType())),
    STACK_DISCOUNT(501,"组套折扣", 2,-1,Integer.valueOf(PromoSubTypeEnum.STACK_DISCOUNT.getPromoSubType())),
    ORDER_RAISE(1001,"订单加价购", 2,-1,Integer.valueOf(PromoSubTypeEnum.ORDER_RAISE.getPromoSubType())),

## PayAssetTypeEnum  实收表记录资产类型

    MONEY_PAY(1001, "货币", 1, 1, 1001,null),
    CASH_COUPON(2001, "代金券", 2, 1,UserCouponTypeEnum.CASH_COUPON.getCode(),null),
    FULL_COUPON(2002, "满减券", 2, 1,UserCouponTypeEnum.FULL_CUT_COUPON.getCode(), SubCouponTypeEnum.SIMPLE.getCode()),
    FREIGHT_COUPON(2003, "运费券", 2, 1,UserCouponTypeEnum.EXPRESS_COUPON.getCode(),null),
    PROCESS_COUPON(2004, "加工券", 2, 1,UserCouponTypeEnum.PROCESS_COUPON.getCode(),null),
    EMPLOYEE_MEAL_COUPON(2005, " 员工餐票", 2, 1,UserCouponTypeEnum.FULL_CUT_COUPON.getCode(),SubCouponTypeEnum.WELFARE.getCode()),
    INTEGRAL(3001, "积分", 2, 1,3001,null),
    BALANCE_PAY(4001, "余额", 1, 1,4001,null),
    E_CARD_PAY(5001, "E卡支付", 1, 1,5001,null);

### 对账状态

    WAIT_RECONCILE(100, "待对账"),
    RECONCILE_COMPLETE(101, "对账完成"),//终态
    RECONCILE_FAIL(104, "对账失败"),//终态
    WRITE_OFF(105, "已核销"),//终态
    REPEAT_PAY(106, "重复支付"),//现阶段出现多支付情况，统一认为是重复支付 //终态
    MULTIPLE_REFUND(108, "多退款"),;//退款时，京东多退钱了 //终态

## 业务流程

![拉平周期系统交互时序](https://raw.githubusercontent.com/unionstars/unionstars.github.io/master/assets/images/pictures/2018-08-07-order-account-business/01-01.png)

## 系统架构

## 现存问题

## 未来规划

## 总结
