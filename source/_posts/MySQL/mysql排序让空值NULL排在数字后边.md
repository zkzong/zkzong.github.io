---
title: MySQL排序让空值NULL排在数字后边
date: 2019-04-19
categories: MySQL
---

从现实项目需求出发；

有一张城市表，里面有北京、上海、广州、河北、天津、河南6座城市。
```
mysql> select * from bjy_order;
+----+------+
| id | city |
+----+------+
|  1 | 北京 |
|  2 | 上海 |
|  3 | 广州 |
|  4 | 河北 |
|  5 | 天津 |
|  6 | 河南 |
+----+------+
```
要求是让上海排第一个，天津排第二个；

最简单粗暴的方法就是添加一个order_number字段，用来标识顺序的；然后通过`order by order_number asc`排序。
```
mysql> select * from bjy_order order by order_number asc;
+----+------+--------------+
| id | city | order_number |
+----+------+--------------+
|  2 | 上海 |            1 |
|  5 | 天津 |            2 |
|  1 | 北京 |            3 |
|  3 | 广州 |            4 |
|  4 | 河北 |            5 |
|  6 | 河南 |            6 |
+----+------+--------------+
```
这么做确实能满足需求。但是如果表里面有中国全部的32个省呢？
再如果来个全国的县市表几百个数据呢？而我们只是想让某几个值排最前面就好了。
就如人们大部分人只知道世界第一高峰是珠穆朗玛峰而不去关注第二第三一样。
我们应该首先想到的就是只给需要排在前面的加上排序数字，其他为NULL。
```
mysql> select * from bjy_order;
+----+------+--------------+
| id | city | order_number |
+----+------+--------------+
|  1 | 北京 | NULL         |
|  2 | 上海 |            1 |
|  3 | 广州 | NULL         |
|  4 | 河北 | NULL         |
|  5 | 天津 |            2 |
|  6 | 河南 | NULL         |
+----+------+--------------+
```
然后我们order by一下；
```
mysql> select * from bjy_order order by order_number asc;
+----+------+--------------+
| id | city | order_number |
+----+------+--------------+
|  1 | 北京 | NULL         |
|  3 | 广州 | NULL         |
|  4 | 河北 | NULL         |
|  6 | 河南 | NULL         |
|  2 | 上海 |            1 |
|  5 | 天津 |            2 |
+----+------+--------------+
```
然而即将成功的时候让人沮丧的事情发生了，那些为NULL的排在在最前面。
OK，下面有请今天的主角出场来解决这个问题。
我们来利用`is null`把sql给稍微改造一下即可。
```
mysql> select * from bjy_order order by order_number is null,order_number asc;
+----+------+--------------+
| id | city | order_number |
+----+------+--------------+
|  2 | 上海 |            1 |
|  5 | 天津 |            2 |
|  1 | 北京 | NULL         |
|  3 | 广州 | NULL         |
|  4 | 河北 | NULL         |
|  6 | 河南 | NULL         |
+----+------+--------------+
```
到此完美实现需求。

