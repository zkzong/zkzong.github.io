## 第1章 Spring之旅

为了降低Java开发的复杂性，Spring采取了以下4种关键策略：

+ 基于POJO的轻量级和最小侵入性编程
+ 通过依赖注入和面向接口实现松耦合
+ 基于切面和管理惯例进行声明式编程
+ 通过切面和模板减少样板式代码

依赖注入方式：

+ 构造器注入
+ setter注入
+ 接口注入（spring不支持）

Spring装配Bean的方式：

+ XML配置

Spring容器类型：

+ Bean工厂（bean factories，由org.springframework.beans.factory.BeanFactory接口定义）
+ **应用上下文**（application由org.springframework.context.ApplicationContext接口定义）
    - ClassPathXmlApplicationContext——从类路径下的XML配置文件中加载上下文定义，把应用上下文定义文件当作类资源
    - FileSystemXmlApplicationContext——读取文件系统下的XML配置文件并加载上下文定义
    - XmlWebApplicationContext——读取Web应用下的XML配置文件并装载上下文定义

## 第2章 装配Bean

实际上，Spring是使用反射来创建Bean的。

所有的Spring Bean默认都是单例。
为了让Spring在每次请求时都为Bean产生一个新的实例，只需要配置Bean的scope属性为prototype即可。
`<bean id="ticket" class="com.zkzong.springinaction.springidol.Ticket" scope="prototype">`

内部Bean

使用Spring的命名空间p装配属性
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
       
<bean id="kenny" class="com.zkzong.springinaction.springidol.Instrumentalist"
    p:song="Jingle Bells"
    p:instrument-ref="piano" />
```

SpEL：Spring表达式语言

## 第3章 最小化Spring XML配置

+ 自动装配
+ 自动检测

### 自动装配

4种类型的自动装配：

+ byName
+ byType
    为了避免因为使用byType自动装配而带来的奇异歧义，Spring提供了另外两种选择：可以为自动装配标识一个首选Bean，或者可以取消某个Bean自动装配的候选资格。
    首选Bean：primary=true。primary属性默认为true，所以需要将所有非首选Bean的primary属性设置为false。
    如果希望排除排除某些Bean，可以autowire设置这些Bean的autowire-candidate属性为false。
+ constructor
+ autodetect

### 注解装配

Spring容器默认禁用注解装配。
启用方式：`<context:annotation-config>`

Spring 3支持几种不同的用于自动装配的注解：

+ Spring自带的@Autowired注解
+ JSR-330的@Inject注解
+ JSR-250的@Resource注解

### 自动检测Bean

`<context:annotation-config>`

### 为自动检测标注Bean
默认情况下，`<context:component-scan>`查找使用构造型（stereotype）注解所标注的类，这些特殊的注解如下：

+ @Component——通用的构造型注解，标识该类为Spring组件
+ @Controller——标识将该类定义为Spring MVC Controller
+ @Repository——标识将该类定义为数据仓库
+ @Service——标识将该类定义为服务
+ 使用@Component标注的任意自定义注解

### 第4章 面向切面的Spring

Spring切面可以应用5种类型的通知：

+ Before —— 在方法被调用之前调用通知
+ After —— 在方法完成之后调用通知，无论方法执行是否成功
+ After-returning —— 在方法成功执行之后调用通知
+ After-throwing —— 在方法抛出异常后调用通知
+ Around —— 通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为

AOP框架：

+ AspectJ
+ JBoss AOP
+ Spring AOP

Spring提供了4种各具特色的AOP支持：

+ 基于代理的经典AOP
+ @AspectJ注解驱动的切面
+ 纯POJO切面
+ 注入式AspectJ切面（适合Spring各版本）

### 第5章 征服数据库

Spring提供了在Spring上下文中配置数据源Bean的多种方式，包括：

+ 通过JDBC驱动程序定义的数据源
+ 通过JNDI查找的数据源（推荐）
+ 连接池的数据源

Spring为JDBC提供了3个模板类供使用：

+ JdbcTemplate
+ NamedParameterJdbcTemplate
+ SimpleJdbcTemplate

#### 在Spring中集成Hibernate

使用Hibernate的主要接口是org.hibernate.Session。Session接口提供了基本的数据访问功能，如保存、更新、删除以及从数据库加载对象的功能。通过Hibernate的Session接口，应用程序的DAO能够满足所有的持久化需求。
获取Hibernate Session对象的标准方式是借助于Hibernate的SessionFactory接口的实现类。除了一些其他的任务，SessionFactory主要负责Hibernate Session的打开、关闭以及管理。
两种配置SessionFactory方式：

+ XML：LocalSessionFactoryBean
+ 注解：AnnotationSessionFactoryBean

#### JPA

+ LocalEntityManagerFactoryBean
+ LocalContainerEntityManagerFactoryBean

为了在Spring中实现EntityManager注入，我们需要在Spring应用上下文中配置一个PersistenceAnnotationBeanPostProcessor：
`<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor" />`

### 第6章 事务管理

ACID：

+ 原子性（Atomic）
+ 一致性（Consistent）
+ 隔离性（Isolated）
+ 持久性（Durable）

Spring提供了对编码式和声明式事务管理的支持。

#### 传播行为
传播规则回答了这样一个问题，即新的事务应该被启动还是被挂起，或者方法是否要在事务环境中运行。
#### 隔离级别
隔离级别定义了一个事务可能受其他并发事务影响的程度。另一种考虑隔离级别的方式就是将其想象成事务对于事务性数据的自私程度。

+ 脏读（Dirty reads）：脏读发生在读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。
+ 不可重复读（Nonrepeatable read）：不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间更新了数据。
+ 幻读（Phantom read）：幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

声明式事务：

+ 在XML中定义事务
    `<tx:advice>`
    `<aop:config>`
+ 定义注解驱动的事务
    `<tx:annotation-driven>`
    `@Transactional`

### Spring MVC

