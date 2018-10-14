---
layout: post
title:  "技术框架及编码规范-技术选型约束"
date:   2018-07-11 12:00:00

categories: Java
tags: learningnote
author: "张学刚"
---

# 手册简介

本文章作为商户财务组，技术框架及编码规范的统一指导规范，其中涉及到的规范和约束，也作为脚手架的约束。规范的约束和脚手架，并不取代部门的规范，而是作为一种增强，对于一些适用的，并未在部门内提供的约束和规范，在组内适用并且有良好效果的，会建议增加到部门脚手架约束和规范中，作为统一标准。这是一系列的标准，本手册，只讨论技术选型的约束。

技术规范的约束，使用的技术栈需要从如下技术栈中选择，选择时，通过starter的方式引用，每个starter提供了开箱即用的组件。

# 技术选型确认

## 基础结构

spring boot / spring mvc / mybatis

## 数据库连接池

HikariCP

## 缓存

caffeine / jimdb

## 消息中间价

JMQ

## 分布式任务

elstic-job / xxl-job（邮件）

## 分库分表

sharding-jdbc/ jed(不需要)

## 监控

UMP

## 远程服务调用

http rest / jsf

## 前端和模板语言

vue.js(不需要) / thymeleaf

## 序列化组件

fastjson

## 配置中心

config

## 批处理

spring batch

## 搜索

elasticsearch

# 编写基于各starter的示例工程

功能名称：xstore-vf-spring-boot

starter名称：xstore-vf-spring-boot-starter-xx 例如：xstore-vf-spring-boot-starter-jsf

使用文档：每个starter的功能及使用方式

## 逐步构建示例工程

## 基于示例工程构建脚手架
