**什么是JPA？**

全称Java Persistence API，可以通过注解或者XML描述【对象-关系表】之间的映射关系，并将实体对象持久化到数据库中。

为我们提供了：

1）**ORM映射元数据：**JPA支持XML和注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中；

如：**@Entity**、**@Table**、**@Column**、**@Transient**等注解。

 2）**JPA 的API：**用来操作实体对象，执行CRUD操作，框架在后台替我们完成所有的事情，开发者从繁琐的JDBC和SQL代码中解脱出来。

如：**entityManager.merge(T t);**

 3）**JPQL查询语言：**通过面向对象而非面向数据库的查询语言查询数据，避免程序的SQL语句紧密耦合。

如：**from Student s where s.name = ?**

但是： 

JPA仅仅是一种规范，也就是说JPA仅仅定义了一些接口，而接口是需要实现才能工作的。所以底层需要某种实现，而Hibernate就是实现了JPA接口的ORM框架。

也就是说：

JPA是一套ORM规范，Hibernate实现了JPA规范！如图：

![JPA规范与ORM框架之间的关系](https://upload-images.jianshu.io/upload_images/292448-feea516e5970b6d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**什么是spring data jpa？**

spirng data jpa是spring提供的一套简化JPA开发的框架，按照约定好的【方法命名规则】写dao层接口，就可以在不写接口实现的情况下，实现对数据库的访问和操作。同时提供了很多除了CRUD之外的功能，如分页、排序、复杂查询等等。

Spring Data JPA 可以理解为 JPA 规范的再次封装抽象，底层还是使用了 Hibernate 的 JPA 技术实现。如图：

![spring data jpa、jpa以及ORM框架之间的关系](https://upload-images.jianshu.io/upload_images/292448-abf082949a72c72a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


接口约定命名规则：

|关键字|方法命名|where条件|
|--|--|--|
|And|findByLastnameAndFirstname|… where x.lastname = ?1 and x.firstname = ?2|
|Or|findByLastnameOrFirstname|… where x.lastname = ?1 or x.firstname = ?2|
|Is,Equals|findByFirstname,findByFirstnameIs,findByFirstnameEquals|… where x.firstname = ?1|
|Between|findByStartDateBetween|… where x.startDate between ?1 and ?2|
|LessThan|findByAgeLessThan|… where x.age < ?1|
|LessThanEqual|findByAgeLessThanEqual|… where x.age <= ?1|
|GreaterThan|findByAgeGreaterThan|… where x.age > ?1|
|GreaterThanEqual|findByAgeGreaterThanEqual|… where x.age >= ?1|
|After|findByStartDateAfter|… where x.startDate > ?1|
|Before|findByStartDateBefore|… where x.startDate < ?1|
|IsNull|findByAgeIsNull|… where x.age is null|
|IsNotNull,NotNull|findByAge(Is)NotNull|… where x.age not null|
|Like|findByFirstnameLike|… where x.firstname like ?1|
|NotLike|findByFirstnameNotLike|… where x.firstname not like ?1|
|StartingWith|findByFirstnameStartingWith|… where x.firstname like ?1 (parameter bound with appended %)|
|EndingWith|findByFirstnameEndingWith|… where x.firstname like ?1 (parameter bound with prepended %)|
|Containing|findByFirstnameContaining|… where x.firstname like ?1 (parameter bound wrapped in %)|
|OrderBy|findByAgeOrderByLastnameDesc|… where x.age = ?1 order by x.lastname desc|
|Not|findByLastnameNot|… where x.lastname <> ?1|
|In|findByAgeIn(Collection<Age> ages)|… where x.age in ?1|
|NotIn|findByAgeNotIn(Collection<Age> ages)|… where x.age not in ?1|
|True|findByActiveTrue()|… where x.active = true|
|False|findByActiveFalse()|… where x.active = false|
|IgnoreCase|findByFirstnameIgnoreCase|… where UPPER(x.firstame) = UPPER(?1)|

实例：

```java
// 遵循命名规范，执行多条件查询
Standard findByNameAndMaxLength(String name, Integer maxLength);
```

```java
// 模糊查询
Standard findByNameLike(String name);
```

spring boot集成spring data jpa只需两步：

第一步：导入maven坐标

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

第二步：yml配置文件中配置jpa信息

```
spring:
    datasource:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/test?useSSL=false&characterEncoding=utf8
      username: root
      password: root

    jpa:
      database: mysql
      show-sql: true
      hibernate:
        ddl-auto: update
```
