---
title: querydsl-4.1.4 translation
date: 2019-10-09 10:29:55
tags:
---

# 1. Introduction 介绍

## 1.1 Background 背景

Querydsl是为满足类型安全的HQL查询而孕育而生。HQL查询的需要大量的字符串拼接和结果集，这会使得代码非常难读。通过简单的字符串来生成的HQL构造的另一个问题是：域类型和属性不安全的引用。

随着域对象类型安全带来大量的益处在软件开中。域改变是通过直接的映射和在查询构造中自动化完成使得查询构造变得更快更安全。

Hibernate的HQL是Querydsl的第一目标语言，但现在已经支持了 JPA, JDO, JDBC, Lucene, Hibernate Search, MongoDB, Collections and RDFBean。

## 1.2 Principles 原则
类型安全是Querydsl的核心，查询的构造是基于通过反射查询类型域类型的属性。方法的调用也是完全安全的。

一致性是另一个重要的原则。查询路径和操作都是相同的基于同一个实现，查询接口也有相同的基础接口。

为了保持Querydsl查询一致的表达性，表达类型可以去查看com.querydsl.core.Query, com.querydsl.core.Fetchable 和 com.querydsl.core.types.Expression这几个类.

# 2.Tutorials 教程
我们提供以整合指导作为主要的Querydsl后端方式，而非一般的集成入门指南。

## 2.1 Querying JPA JPA查询



