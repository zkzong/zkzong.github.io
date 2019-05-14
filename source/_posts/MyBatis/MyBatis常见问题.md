---
title: MyBatis常见问题
date: 2019-05-14
categories: MyBatis
---

```
@Select("select * from user where name = #{name}")
User findByName(String name);

// 两个参数必须加@Param注解
@Select("select * from user where name = #{name} and age = #{age}")
User findByNameAndAge(@Param("name") String name, @Param("age") Integer age);
```
如果只有一个参数，可以不加@Param注解，但是如果参数大于1个，必须加@Param注解，否则报错。
    
@Select动态sql

[https://segmentfault.com/q/1010000006875476](https://segmentfault.com/q/1010000006875476)
