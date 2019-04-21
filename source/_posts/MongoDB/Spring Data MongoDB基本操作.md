---
title: Spring Data MongoDB基本操作
date: 2019-04-21
categories: MongoDB
---

本文主要借号如何使用Spring Data MongoDB进行CRUD操作，有两种方式：注解方式和XML方式。

## 1. 引入依赖

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>1.10.7.RELEASE</version>
</dependency>
```

## 2. 使用注解和XML配置

### 2.1 注解

继承`AbstractMongoConfiguration`是最快的方式，它配置了所有你需要的配置，如`mongoTemplate`。

```
import com.mongodb.Mongo;
import com.mongodb.MongoClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.config.AbstractMongoConfiguration;

@Configuration
public class SpringMongoConfig extends AbstractMongoConfiguration {
    @Override
    protected String getDatabaseName() {
        return "test";
    }

    @Bean
    @Override
    public Mongo mongo() throws Exception {
        return new MongoClient("127.0.0.1");
    }
}
```

但是，下面这种配置更灵活。

```
import com.mongodb.MongoClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.MongoDbFactory;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoDbFactory;

import java.net.UnknownHostException;

@Configuration
public class SpringMongoConfig1 {
    @Bean
    public MongoDbFactory mongoDbFactory() throws UnknownHostException {
        return new SimpleMongoDbFactory(new MongoClient(), "test");
    }

    @Bean
    public MongoTemplate mongoTemplate() throws UnknownHostException {
        MongoTemplate mongoTemplate = new MongoTemplate(mongoDbFactory());
        return mongoTemplate;
    }
}
```

使用`AnnotationConfigApplicationContext`加载配置：
```
ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringMongoConfig.class);
MongoOperations mongoOperation = (MongoOperations)ctx.getBean("mongoTemplate");
```

### 2.2 XML

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mongo="http://www.springframework.org/schema/data/mongo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/data/mongo
       http://www.springframework.org/schema/data/mongo/spring-mongo.xsd">

    <mongo:mongo host="127.0.0.1" port="27017" />
    <mongo:db-factory dbname="test" />

    <bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
        <constructor-arg name="mongoDbFactory" ref="mongoDbFactory" />
    </bean>
</beans>
```

使用`GenericXmlApplicationContext`加载配置：
```
ApplicationContext ctx = new GenericXmlApplicationContext("SpringConfig.xml");
MongoOperations mongoOperation = (MongoOperations)ctx.getBean("mongoTemplate");
```

## 3. User实体类

```
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "users")
public class User {

	@Id
	private String id;

	String username;

	String password;

	//getter, setter, toString, Constructors

}
```

## 4. 实例 - CRUD操作

```java
package com.zkzong.mongodb.springdata.core;

import com.zkzong.mongodb.springdata.config.SpringMongoConfig;
import com.zkzong.mongodb.springdata.model.User;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;

import java.util.List;

public class App {
    public static void main(String[] args) {
        // For XML
        //ApplicationContext ctx = new GenericXmlApplicationContext("SpringConfig.xml");

        // For Annotation
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringMongoConfig.class);
        MongoOperations mongoOperation = (MongoOperations) ctx.getBean("mongoTemplate");

        User user = new User("zong", "zong");

        // save
        mongoOperation.save(user);

        // now user object got the created id.
        System.out.println("1. user : " + user);

        // query to search user
        Query searchUserQuery = new Query(Criteria.where("username").is("zong"));

        // find the saved user again.
        User savedUser = mongoOperation.findOne(searchUserQuery, User.class);
        System.out.println("2. find - savedUser : " + savedUser);

        // update password
        mongoOperation.updateFirst(searchUserQuery, Update.update("password", "new password"), User.class);

        // find the updated user object
        User updatedUser = mongoOperation.findOne(searchUserQuery, User.class);

        System.out.println("3. updatedUser : " + updatedUser);

        // delete
        mongoOperation.remove(searchUserQuery, User.class);

        // List, it should be empty now.
        List<User> listUser = mongoOperation.findAll(User.class);
        System.out.println("4. Number of user = " + listUser.size());

    }
}
```
