---
layout: post
title:  "Celan 软件架构面临的问题"
date:   2018-07-11 12:00:00

categories: tool
tags: archtecture java tool
author: "XueGang Zhang"
image: /images/logo.png
comments: true
published: true
---

#### 软件架构面临的问题

Bob在在整洁架构中提到：`软件架构的终极目标是，用最小的人力成本来满足构建和维护系统的需求` ，Eric Evans在领域驱动设计中也多次强调架构设计的重要。我们也常常看到的很多三年以上的工程领域模型混乱、架构分层不合理、项目工程边界划分混乱、重复功能多。导致这种现象的原因很多，最终的原因可能是人的问题，人的软件水准参差不齐，人员流动大等等很多元素，在我跟很多大厂的朋友聊天，他们也一致的反馈了很多一致的结果，业务系统代码的逐步腐化，很整洁的业务系统很少。

更多的，我们看到很多这样的现象，一个技术复杂度简单的系统，却需要很多人去维护，对于开发者而言要7*24小时的救火，对于管理者来言，简单的功能上线的投入越来越大，如何规避这种问题呢? 上面我们说这些问题是人引起的，那解决这些问题就要在人上去解决，如果你用心发现，你会发现这样一个问题，团队与团队之间的差距是巨大的，如果一个学习氛围浓厚的团队，他们的技术输出能力，沉淀能力与其它团队相比差距是巨大的。

是的，我要聊的就是建立大家的软件的审美，建立大家的热情，建立共同的约束、共同前行沉淀文化。我一直认为写代码、写作和绘画，所有这些创造的过程是基本类似的，第一位的是你要有正确的审美，第二你要有创造的热情，第三制定规约砥砺前行。最后慢慢沉淀思想的结晶和技术文化。人员的流动是控制不了的，但是思想的沉淀却如同给后边的人留下了巨人的肩膀。


#### 如何解决这些问题

如何解决这些人的问题呢，我们可以通过招聘去招比自己更优秀的人，通过文化思想的相荡来提升大家的审美，也可以内部培养，但是更多的时候，这需要长期的建设，如何短时间内就形成一种积累呢，那就是工程纪律的建设。工程纪律可以参考性质的，比如阿里出的java开发手册以及附带的idea的检查工具；也可以是强制性的，比如在代码合并的时候增加代码规则检查，如果不通过不能进行分支合并或者上线。我们通过大量的实施发现这些宽松的或者强制的检查，虽然初期可能是不适的，但是日益积累对技术人员的影响却是巨大的，他实实在在影响了技术文化。


#### 工程纪律的实施

我们发现一个问题，如果你让研发人原去看文档，告诉他在执行书写mybatis的mapper的文件的时候，数据库的关键字习惯用大写，表名和字段名习惯小写；常量的定义习惯用大写并用下划线进行分隔，远远不如给他快速生成一个约定俗成的工程框架并赋以例子来的实在。

下面是脚手架实现的约束内容，有些是通过例子来约束，有些通过内置的组件进行约束。


#### 工程创建

架手架提供了三套工程的创建模板，中心端工程（服务端的工程）、API工程、网关工程， 按用户为例，我们可以创建一个用户中心的后端服务工程、创建一个基于RPC框架的远程调用接口工程、创建一个Restful风格的http网关工程。这是我创建好的三个工程名称：`user-api  user-center  user-gateway` 而这非常简单。这里的分包原则用了传统的分层级架构，这种分层架构可能不是最合理的，可能在构建复杂系统的时候也存在很多问题，包括现在微软的.net云案例也开始使用DDD的框架结构进行构建，但是这种分层在短时间内更容易学习，不过随着技术文化的沉淀，大家对领域驱动设计实现的熟练会逐步改变，在这里更主要的让大家了解DDD的思想。

user-api
├── pom.xml
└── src
    └── main
        └── java
            └── com
                └── unionstars
                    └── user
                        └── api
                            ├── Request.java
                            ├── Response.java
                            ├── facade
                            │   └── UserServiceProvider.java
                            └── mo
                                └── UserMO.java


user-center
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── unionstars
│   │   │           └── user
│   │   │               └── center
│   │   │                   ├── Application.java
│   │   │                   ├── common
│   │   │                   │   └── GsonUtil.java
│   │   │                   ├── configration
│   │   │                   ├── controller
│   │   │                   │   ├── IndexViewController.java
│   │   │                   │   └── LoginViewController.java
│   │   │                   ├── dao
│   │   │                   │   └── UserMapper.java
│   │   │                   ├── domain
│   │   │                   │   └── User.java
│   │   │                   ├── interceptor
│   │   │                   │   └── LoginInterceptor.java
│   │   │                   └── service
│   │   │                       ├── UserService.java
│   │   │                       └── impl
│   │   │                           └── UserServiceImpl.java
│   │   └── resources
│   │       ├── application-dev.yml
│   │       ├── application.yml
│   │       ├── db
│   │       │   └── schema.sql
│   │       ├── logback.groovy
│   │       ├── mapper
│   │       │   └── UserMapper.xml
│   │       ├── spring
│   │       │   ├── spring-config.xml
│   │       │   ├── spring-jsf-consumer.xml
│   │       │   └── spring-jsf-provider.xml
│   │       ├── static
│   │       │   ├── css
│   │       │   │   └── login
│   │       │   │       └── login.css
│   │       │   ├── image
│   │       │   │   └── login
│   │       │   │       ├── error.png
│   │       │   │       ├── pwd.png
│   │       │   │       └── user.png
│   │       │   └── js
│   │       │       └── login
│   │       │           ├── jquery
│   │       │           │   └── jquery.min.js
│   │       │           └── md5
│   │       │               └── md5.js
│   │       └── templates
│   │           ├── login.html
│   │           └── welcome.html
│   └── test
│       └── java
│           └── com
│               └── unionstars
│                   └── user
│                       └── center
│                           ├── BaseTest.java
│                           └── service
│                               └── UserServiceTest.java


user-gateway
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── unionstars
    │   │           └── user
    │   │               └── gateway
    │   │                   ├── RestApplication.java
    │   │                   ├── config
    │   │                   │   ├── BaseResult.java
    │   │                   │   └── SwaggerConfig.java
    │   │                   ├── controller
    │   │                   │   └── UserController.java
    │   │                   └── model
    │   │                       └── User.java
    │   └── resources
    │       ├── application-dev.yml
    │       ├── application.yml
    │       ├── logback.groovy
    │       └── spring
    │           ├── spring-config.xml
    │           ├── spring-jsf-consumer.xml
    │           └── spring-jsf-provider.xml
    └── test
        └── java
            └── com
                └── unionstars
                    └── user
                        └── gateway
                            ├── BaseTest.java
                            └── controller
                                └── UserControllerTest.java
##### 

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

#### 规范中默认约束的组件和版本

| 基础组件    | 版本          | 自研组件 | 版本  |
| ----------- | ------------- | -------- | ----- |
| spring-boot | 2.1.7.RELEASE | jsf      | 1.6.9 |
| jdk         | 1.8           |          |       |
| lombok      | 1.18.8        |          |       |
| gson        | 2.8.5         |          |       |


#### 脚手架!!!

现在基于以上的原则，我们可以迅速生成一个开箱即用的工程，参考本文档你仅仅需要几步就可以生成一个开箱即用的工程，并且提供了多种工程模板，api工程、rest风格的工程、center工程。[开始吧](https://www.npmjs.com/package/generator-unionstars)




