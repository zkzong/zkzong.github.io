---
title: MySQL查询结果-1变为true
date: 2019-04-19
categories: MySQL
---

使用sql查询出某个字段的值为1，然后转换成Map< String, Object >，此时该字段值变为true，而不是原来的1。
原因：数据库中该字段定义的类型为tinyint
解决办法：
1. 修改该字段的定义类型：使用非tinyint类型
2. 修改sql语句，使用CONCAT函数处理该字段：CONCAT(fieldName, '')。
