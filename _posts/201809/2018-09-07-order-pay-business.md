---
layout: post
title:  "订单支付业务流程"
date:   2018-07-11 12:00:00

categories: Java
tags: learningnote
author: "张学刚"
---

`此文件描述核心业务`

# 订单支付中心

>A Pay center for order.

## 业务概述

台帐就是明细纪录表。台帐，不属于会计核算中的帐簿系统，不是会计核算时所记的帐簿，它是企业为了加强某方面的管理、更加详细地了解某方面的信息而设置的一种辅助帐簿，
没有固定的格式，没有固定的帐页，企业可根据实际需要自行设计，尽量详细，以全面反映某方面的信息，不必按凭证号记帐，但能反映出记帐号更好。具体在仓库管理中，台账详细记录了什么时间，
什么仓库，什么货架由谁入库了什么货物，共多少数量。同样，出库也要对应出库台账。再说的明白一点，台账就是流水账。

订单支付系统针对货币支付的内容对接响应的第三方支付或者聚合支付，完成对订单的收款，完成收款后，通知订单
对账完成后，订单中间件(xstore-order-middleware)接收对账结果，并下发给OPC(订单生产中心)控制订单生产。****

## 名词解释

### PaySaaSChannelEnumMO 支付渠道枚举

    JD_CODE_CHANNEL(5, "京东扫码支付"),
    WX_CODE_CHANNEL(6, "微信刷卡支付"),
    FACE_CHANNEL(10, "刷脸支付"),
    JD_STAFFCARD_CHANNEL(13, "员工卡支付"),
    JD_STAFFCARD_M1(20, "M1员工卡支付"),
    POS_CARD(7, "银行卡支付"),
    CASH(8, "现金支付"),
    WX_NP(3, "微信免密支付"),

### PayTool 支付工具

    JD_APP("1", "jdApp", 1, 1, "京东app支付","JD"),
    WX_APP("2", "wxApp", 2, 1, "微信app支付","WX"),
    WX_NP("3", "wxNp", 3, 1, "微信免密支付","WX"),
    JD_NP("4", "jdNp", 4, 1, "京东免密支付","JD"),
    JD_CODE("5", "jdCode", 5, 1, "京东付款码支付","JD"),
    WX_CODE("6", "wxCode", 6, 1, "微信付款码支付","WX"),
    POS_CARD("7", "posCard", 7, 2, "pos刷卡支付","BANK"),
    CASH("8", "cash", 8, 2, "现金支付","BANK"),
    MEMBER_CARD("9", "memberCard", 9, 2, "会员卡支付","BANK"),
    BRUSH_FACE("10", "brushFace", 10, 1, "刷脸支付","JD"),
    WX_QRCODE("11", "wxQrcode", 11, 1, "微信二维码支付","WX"),
    JD_QRCODE("12", "jdQrcode", 12, 1, "京东二维码支付","JD"),
    JD_STAFFCARD("13", "staffcard", 13, 1, "员工卡支付","JD"),
    WX_H5("14", "wxH5", 14, 1, "微信h5支付","WX"),
    JD_H5("15", "jdH5", 15, 1, "京东h5支付","JD"),
    WX_OFFICIALACCOUNTS("16", "officialaccounts", 16, 1, "微信公众号支付","WX"),
    WIRETRANSFER("17", "wiretransfer", 17, 2, "电汇","BANK"),
    CHECK("18", "check", 18, 2, "支票","BANK"),
    WX_MINIPROGRAM("19", "wxMiniProgram", 19, 1, "微信小程序支付", "WX"),
    JD_STAFFCARD_M1("20", "staffcard_m1", 20, 1, "实体员工卡支付", "JD"),

### MerchantEnum 签约商户枚举

    WX("0", "WX", 1, "微信商户"),
    JD("1", "JD", 2, "京东商户"),

### RefundTypeEnum 退款类型枚举

    ORDER(1000, "订单"),
    AFTER_SALE(1001, "售后服务单"),
    ADJUSTMENT(1002, "调整单"),
    COMPENSATE(1003, "赔付单"),
    CANCEL(1004, "售前取消订单"),

### TradeSourceEnum 系统级别交易来源

    FRESH("0","fresh","1","7fresh"),
    NOBODYSTORE("1","nobody","2","无人店");

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
