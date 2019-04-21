---
title: CrudRepository JpaRepository PagingAndSortingRepository之间的区别
date: 2019-04-21
categories: Spring
---

## 1. 简介

本文介绍三种不同的Spring Data repository和它们的功能，包含以下三种：
* CrudRepository
* PagingAndSortingRepository
* JpaRepository
简单地说，[Spring Data](http://spring.io/projects/spring-data)中的每个repository都继承自`Repository`接口，但是，除此之外，它们每个又有不同的功能。

## 2. Spring Data Repositories

首先介绍`JpaRepository`，它继承自`PagingAndSortingRepository`，而`PagingAndSortingRepository`又继承自`CrudRepository`。
每个都有自己的功能：
* CrudRepository提供CRUD的功能。
* PagingAndSortingRepository提供分页和排序功能
* JpaRepository提供JPA相关的方法，如刷新持久化数据、批量删除。

由于三者之间的继承关系，所以**JpaRepository包含了CrudRepository和PagingAndSortingRepository所有的API**。

当我们不需要JpaRepository和PagingAndSortingRepository提供的功能时，可以简单使用CrudRepository。

下面我们通过例子更好地理解它们提供的API。
首先创建Product实体：
```java
@Entity
public class Product {
 
    @Id
    private long id;
    private String name;
 
    // getters and setters
}
```
然后实现通过名称查询Product的操作：
```java
@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    Product findByName(String productName);
}
```
这样就实现了通过名称查询Product的方法，Spring Data Repository根据方法名自动生成相应的实现。

## 3. CrudRepository

先看下`CrudRepository`接口的源码：
```java
public interface CrudRepository<T, ID extends Serializable>
  extends Repository<T, ID> {
 
    <S extends T> S save(S entity);
 
    T findOne(ID primaryKey);
 
    Iterable<T> findAll();
 
    Long count();
 
    void delete(T entity);
 
    boolean exists(ID primaryKey);
}
```
方法功能介绍：
+ save(...) – 保存多个对象。这里，我们可以传多个对象批量保存。
+ findOne(...) – 根据传入的主键值获取单一对象。
+ findAll(...) – 查询数据库中所有对象。
+ count() – 返回表中记录总数。
+ delete(...) – 根据传入的对象删除记录。
+ exists(...) – 根据传入的主键值判断对象是否存在。
CrudRepository接口看起来非常简单，但实际上它提供了所有基本查询保存等操作。

## 4. PagingAndSortingRepository

PagingAndSortingRepository接口的源码如下：
```java
public interface PagingAndSortingRepository<T, ID extends Serializable> 
  extends CrudRepository<T, ID> {
 
    Iterable<T> findAll(Sort sort);
 
    Page<T> findAll(Pageable pageable);
}
```
该接口提供了`findAll(Pageable pageable)`这个方法，它是实现分页的关键。
使用Pageable时，需要创建Pageable对象并至少设置下面3个参数：
+ 1、 每页包含记录数
+ 2、 当前页数
+ 3、 排序
假设我们要显示第一页的结果集，一页不超过5条记录，并根据`lastName`升序排序，下面代码展示如何使用PageRequest和Sort获取这个结果：
```java
Sort sort = new Sort(new Sort.Order(Direction.ASC, "lastName"));
Pageable pageable = new PageRequest(0, 5, sort);
```

## 5. JpaRepository

JpaRepository接口源码：
```java
public interface JpaRepository<T, ID extends Serializable> extends
  PagingAndSortingRepository<T, ID> {
 
    List<T> findAll();
 
    List<T> findAll(Sort sort);
 
    List<T> save(Iterable<? extends T> entities);
 
    void flush();
 
    T saveAndFlush(T entity);
 
    void deleteInBatch(Iterable<T> entities);
}
```
简单介绍下每个方法的作用：
+ findAll() – 获取数据库表中所有实体。
+ findAll(…) – 根据条件获取排序后的所有实体。
+ save(…) – 保存多个实体。这里可以传入多个对象批量保存。
+ flush() – 将所有挂起的任务刷新到数据库。
+ saveAndFlush(…) – 保存实体并实时刷新到数据库。
+ deleteInBatch(…) – 删除多个实体。这里可以传入多个对象并批量删除。
显然，JpaRepository接口继承自PagingAndSortingRepository，PagingAndSortingRepository继承自CrudRepository，所以JpaRepository拥有CrudRepository所有的方法。

## 6. Spring Data Repositories的缺点

尽管Spring Data Repositories有很多优点，但是它们也有缺点：
1. 这种方式把我们的代码和类库以及具体抽象（如：Page和Pageable）绑定。它们不是这个类库独有的，但是我们必须小心不要暴露这些内部实现的细节。
2. 通过继承CrudRepository接口，我们暴露了所有方法。这可能满足大部分情况，但是我们可能会遇到这样的情况：我们希望获得对公开的方法的更细粒度的控制，例如创建一个不包含save(...)和delete(...)的ReadOnlyRepository，此时继承CrudRepository就不满足要求了。


参考文献：
[CrudRepository, JpaRepository, and PagingAndSortingRepository in Spring Data](https://www.baeldung.com/spring-data-repositories)
