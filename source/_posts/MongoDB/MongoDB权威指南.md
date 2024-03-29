---
title: MongoDB权威指南
date: 2019-04-21
categories: MongoDB
---

# 第1章 简介
MongoDB是面向文档的数据库。

优点：
1. 丰富的数据类型
2. 容易扩展
3. 丰富的功能
4. 索引
> - 索引
> - 存储JavaScript
> - 聚合
> - 固定集合
> - 文件存储
5. 不牺牲速度
6. 简便的管理

# 第2章 入门
基本概念：
1. 文档是MongoDB中数据的基本单位，非常类似于关系数据库管理系统中的行（但是比行要复杂得多）。
2. 类似地，集合可以被看作是没有模式的表。
3. MongoDB的单个实例可以容纳多个独立的数据库，每一个都有自己的集合和权限。
4. MongoDB自带简洁但功能强大的JavaScript shell，这个工具对于管理MongoDB实例和操作数据作用非常大。
5. 每一个文档都有一个特殊的键“_id”，它在文档所处的集合中是唯一的。

## 2.1 文档
多个键及其关联的值有序地放置在一起便是文档。

+ 文档中的键/值对是有序的。
+ 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型。文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。
+ 键不能含有\0（空字符）。这个字符用来表示键的结尾。
+ .和$有特别的意义，只有在特定环境下才能使用。
+ 以下划线“_”开头的键是保留的。
+ MongoDB不但区分类型，也区分大小写。
+ MongoDB的文档不能有重复的键。

## 2.2 集合
集合就是一组文档。
         
### 2.2.1 无模式
集合是无模式的。这意味着一个集合里面的文档可以是格式各样的。

### 2.2.2 命名
集合名可以是满足下列条件的任意UTF-8字符串：

+ 集合名不能是空字符串""。
+ 集合名不能含有\0字符（空字符，这个字符表示集合名的结尾。
+ 集合名不能以"system."开头，这是为系统集合保留的前缀。
+ 用户创建的集合名字不能含有保留字符$。

# 2.3 数据库
MongoDB中多个文档组成集合，同样多个集合可以组成数据库。一个MongoDB实例可以承载多个数据库，他们之间可视为完全独立的。每个数据库都有独立的权限控制。

数据库通过名字来标识。数据库名可以是满足以下条件的任意UTF-8字符串。

+ 不能是空字符串（""）。
+ 不得含有''（空格）、.、$、/、\和\0（空字符）。
+ 应全部小写。
+ 最多64字节。
要记住一点，数据库名最终会变成文件系统里的文件。

## 2.4 启动MongoDB
Linux：./mongod
Windows：mongod.exe

## 2.5 MongoDB shell
Linux：./mongo
Windows：mongo

**shell中的基本操作**
创建：insert
读取：find、findOne
更新：update
删除：remove

**使用shell的窍门**
help
