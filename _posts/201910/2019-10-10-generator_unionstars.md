---
layout: post
title:  "Celan 工程纪律的保证"
date:   2018-07-11 12:00:00

categories: tool
tags: archtecture java tool
author: "XueGang Zhang"
image: /images/logo.png
comments: true
published: true
---

#### 背景介绍

很多团队在实施的过程中都没有能够建立起这样的工程纪律，最后造成代码结构的混乱。本文涉及一些最佳实践的规约以及工具的支持，帮助开发者去建造整洁工程，但是团队技术的建设，通过规约和工具只是一个辅助作用，更多的是技术文化的建设，来提高大家对代码审美的能力，通过意识的转变以及相互影响来刺激团队的持续增长。

本文主要涉及如下内容，先从整洁架构说起，然后再从具体的场景下如何对架构进行选型，最后对选型的架构如何实施进行阐述，其中还涉及了一些开发中经常遇到的工程规范。


#### 架构分层





#### mysql规范

##### 库的字符集必须使用utf8
```sql
  CREATE DATABASE  DBORDER CHARACTER SET utf8 ;
  USE DBORDER;
```
##### 创建表的使用语句

```sql
CREATE TABLE PAGE (
  `id` bigint(20)  unsigned   NOT NULL AUTO_INCREMENT COMMENT '自增id',
  `url` varchar(255)  NOT NULL COMMENT '页面地址',
  `description` varchar(255)  DEFAULT NULL COMMENT '页面描述',
  `created` datetime NOT NULL COMMENT '创建时间',
  `modified` datetime NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`)
) ENGINE=Innodb AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT="记录操作人信息";
```
`
注意:
有无符号:unsigned必须填；
主键自增:AUTO_INCREMENT必填;
表的引擎:ENGINE=Innodb必填;
自增起值:AUTO_INCREMENT=xx必填;
表字符集:CHARSET=utf8必填;
表的注释:COMMENT="记录操作人信息"必填;
字段注释:COMMENT="自增id"必填;
`

##### 增加字段时COMMENT必须填写
```sql
ALTER TABLE PAGE ADD COLUMN OPERATOR VARCHAR (20) NULL COMMENT '仓库操作人';
```

##### 索引名称必须以idx开头， 唯一索引用uniq_开头
```sql
ALTER TABLE PAGE ADD INDEX IDX_OPERATOR(OPERATOR);
```

##### mybatis配置中习惯关键字大写，表名库名小写


#### ArchUnit与generator-unionstars



减少工程创建时大量的重复工作、统一代码风格、提供开发样例、对maven仓库的依赖进行统一约束，并提供非springboot生态下的自研中间件的starter组件，以实现完整的开箱即用体验。

## 规范中默认约束的组件和版本

| 基础组件    | 版本          | 自研组件 | 版本  |
| ----------- | ------------- | -------- | ----- |
| spring-boot | 2.1.7.RELEASE | jsf      | 1.6.9 |
| jdk         | 1.8           |          |       |
| lombok      | 1.18.8        |          |       |
| gson        | 2.8.5         |          |       |



