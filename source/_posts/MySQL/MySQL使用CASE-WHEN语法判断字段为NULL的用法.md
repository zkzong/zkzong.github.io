---
title: MySQL使用CASE-WHEN语法判断字段为NULL的用法
date: 2019-04-19
categories: MySQL
---

在写sql语句时，遇到比较复杂的sql可能经常会用到`CASE WHEN`判断，`CASE WHEN`的基本语法在此不再赘述，网上有许多相关教程。

**数据准备：**
```sql
DROP TABLE IF EXISTS `t_user`;
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(10) NOT NULL,
  `sex` smallint(1) NOT NULL,
  `email` varchar(40) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8;

INSERT INTO t_user(name, sex, email) VALUES ('zong', '0', 'zong@163.com');
INSERT INTO t_user(name, sex, email) VALUES ('liu', '1', null);
INSERT INTO t_user(name, sex, email) VALUES ('ma', '1', null);
INSERT INTO t_user(name, sex, email) VALUES ('wang', '0', 'wang@163.com');
INSERT INTO t_user(name, sex, email) VALUES ('zhao', '0', null);
INSERT INTO t_user(name, sex, email) VALUES ('li', '1', null);
```

**简单使用：**
```sql
SELECT name, CASE sex WHEN 0 THEN '男' WHEN 1 THEN '女' ELSE '' END FROM t_user;
SELECT name, CASE sex WHEN 0 THEN '男' WHEN 1 THEN '女' ELSE '' END sex FROM t_user;
```

**判断email是否为null：**
```sql
SELECT 
	name, 
	CASE WHEN email IS NULL THEN '' ELSE email END email
FROM t_user;
```
