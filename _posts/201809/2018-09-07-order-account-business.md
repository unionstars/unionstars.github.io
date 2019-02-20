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

台帐，不属于会计核算中的帐簿系统，不是会计核算时所记的帐簿，它是企业为了加强某方面的管理、更加详细地了解某方面的信息而设置的一种辅助帐簿，没有固定的格式，没有固定的帐页，企业可根据实际需要自行设计，尽量详细，以全面反映某方面的信息，不必按凭证号记帐，但能反映出记帐号更好。

具体在仓库管理中，台账详细记录了什么时间，什么仓库，什么货架由谁入库了什么货物，共多少数量。同样，出库也要对应出库台账。我们可以看到，再说的明白一点，台账就是流水账。

回到我们的订单台账系统，他主要记录的是订单销售的应收金额(`下单后，用户需要支付给商城的金额: SKU的实际金额+运费+服务费-优惠`)流水，订单销售的实收金额(`下单时或者下单后，用户已支付给京东的金额，包括虚拟资产（优惠券、京豆、余额、礼品卡）和在线支付等。`)流水，然后比较应收的金额和实收的金额，确保订单账款无误后，触发订单进行生产。

## 名词解释

因为是以产品化、组件化的理论创建，每个系统都可以平台无关，针对这种属性，对系统架构设计上也引入了一些，平台化的属性，如果认为每个系统的提供的功能是一块积木，那各个积木块的接口，它们是公共开放的。

订单来源(OrderSource)：理论上订单台账系统，平台无关，他可以接受任意平台的应收、实收流水数据，并进行对账，确保账款无误后，通过事件触发订单生产。他的示例值：京东、天猫、淘宝、美团。

原始单据编号(originalBillId): 原始单据编号，正向的时候传订单编号，逆向的时候传服务单号，这个地方主要是因为，售后服务单、调整单、取消单所对应的订单号，都是一致的，所以这个地方记录售后服务单号、调整单号、取消单号，订单编号用(orderId)进行存储。

| 原始单据编号 | 订单编号 |
| ------------ | -------- |
| 0000001      | 1000001  |
| 0000002      | 1000001  |

业务类型(businessType)

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
