---
title: MyBatis中resultType和resultMap的区别
date: 2019-05-23
categories: MyBatis
---

使用MyBatis查询数据库记录时，返回类型常用的有两种：resultType和resultMap。那么两者之间有什么区别呢？

如果只是返回一个值，比如说String或者int，那直接用resultType就行了，`resultType="java.lang.String"`。
```xml
<select id="getUserName" resultType="java.lang.String">
    select user_name from t_users
</select>
```

如果sql查询结果返回的列名和实体类中的字段名一致，可以使用resultType，MyBatis会自动把查询结果赋值给和字段名一致的字段。
**实体类对象：**
```java
@Data
public class Users {
    
    private Long id;
    private String userName;
    private Integer sex;

}
```
**Mapper：**
```xml
<select id="getUsersType" resultType="com.zkzong.mybatis.domain.Users">
    select user_name userName, sex from t_users
</select>
```
如果不一致，sql语句中可以使用别名的方式使其一致。

当sql的列名和实体类的列名不一致，这时就可以使用resultMap了。
```xml
<resultMap id="userMap" type="com.zkzong.mybatis.domain.Users">
    <id property="id" column="id"/>
    <result property="userName" column="user_name"/>
    <result property="sex" column="sex"/>
</resultMap>

<select id="getUsersMap" resultMap="userMap">
    select id, user_name, sex from t_users
</select>
```
property是实体类的字段名，column是sql查询的列名。

对于简单的映射，resultType和resultMap区别不大。但是resultMap功能更强大，可以通过设置typeHander来自定义实现功能。