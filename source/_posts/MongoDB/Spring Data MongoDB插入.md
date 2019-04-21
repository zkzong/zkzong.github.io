---
title: Spring Data MongoDB插入
date: 2019-04-21
categories: MongoDB
---

在Spring Data MongoDB中，使用`save()`和`insert()`方法可以插入一个或多个文档到数据库。

```
User user = new User("...");

//save user object into "user" collection / table
//class name will be used as collection name
mongoOperation.save(user);

//save user object into "tableA" collection
mongoOperation.save(user,"tableA");

//insert user object into "user" collection
//class name will be used as collection name
mongoOperation.insert(user);

//insert user object into "tableA" collection
mongoOperation.insert(user, "tableA");

//insert a list of user objects
mongoOperation.insert(listofUser);
```
默认情况下，保存文档时如果没有指定集合名，使用类名作为集合名。

## 1. 插入

save和insert的区别：

+ 1. save - 应该叫做saveOrUpdate()，当`_id`存在执行`insert()`，当`_id`不存在执行`update()`。
+ 2. insert - 只是insert，如果`_id`存在会报错。

```
//get an existed data, and update it
User userD1 = mongoOperation.findOne(
	new Query(Criteria.where("age").is(64)), User.class);
userD1.setName("new name");
userD1.setAge(100);

//if you insert it, 'E11000 duplicate key error index' error is generated.
//mongoOperation.insert(userD1);

//instead you should use save.
mongoOperation.save(userD1);
```

## 2. 插入文档完整实例

使用spring容器创建mongoTemplate。
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.core.MongoTemplate;
import com.mongodb.MongoClient;

/**
 * Spring MongoDB configuration file
 *
 */
@Configuration
public class SpringMongoConfig{

	public @Bean
	MongoTemplate mongoTemplate() throws Exception {

		MongoTemplate mongoTemplate =
			new MongoTemplate(new MongoClient("127.0.0.1"),"yourdb");
		return mongoTemplate;

	}

}
```

使用`@Document`指定集合名称。本例中，保存user对象时，保存的集合名称是`users`。

```
import java.util.Date;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.format.annotation.DateTimeFormat.ISO;

@Document(collection = "users")
public class User {

	@Id
	private String id;

	@Indexed
	private String ic;

	private String name;

	private int age;

	@DateTimeFormat(iso = ISO.DATE_TIME)
	private Date createdDate;

	//getter and setter methods
}
```

```
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;

import com.mkyong.config.SpringMongoConfig;
import com.mkyong.user.User;

public class App {

	public static void main(String[] args) {
		// For Annotation
		ApplicationContext ctx =
                     new AnnotationConfigApplicationContext(SpringMongoConfig.class);
		MongoOperations mongoOperation =
                     (MongoOperations) ctx.getBean("mongoTemplate");

		// Case1 - insert a user, put "tableA" as collection name
		System.out.println("Case 1...");
		User userA = new User("1000", "apple", 54, new Date());
		mongoOperation.save(userA, "tableA");

		// find
		Query findUserQuery = new Query();
		findUserQuery.addCriteria(Criteria.where("ic").is("1000"));
		User userA1 = mongoOperation.findOne(findUserQuery, User.class, "tableA");
		System.out.println(userA1);

		// Case2 - insert a user, put entity as collection name
		System.out.println("Case 2...");
		User userB = new User("2000", "orange", 64, new Date());
		mongoOperation.save(userB);

		// find
		User userB1 = mongoOperation.findOne(
                     new Query(Criteria.where("age").is(64)), User.class);
		System.out.println(userB1);

		// Case3 - insert a list of users
		System.out.println("Case 3...");
		User userC = new User("3000", "metallica", 34, new Date());
		User userD = new User("4000", "metallica", 34, new Date());
		User userE = new User("5000", "metallica", 34, new Date());
		List<User> userList = new ArrayList<User>();
		userList.add(userC);
		userList.add(userD);
		userList.add(userE);
		mongoOperation.insert(userList, User.class);

		// find
		List<User> users = mongoOperation.find(
                           new Query(Criteria.where("name").is("metallica")),
			   User.class);

		for (User temp : users) {
			System.out.println(temp);
		}

		//save vs insert
		System.out.println("Case 4...");
		User userD1 = mongoOperation.findOne(
                          new Query(Criteria.where("age").is(64)), User.class);
		userD1.setName("new name");
		userD1.setAge(100);

		//E11000 duplicate key error index, _id existed
		//mongoOperation.insert(userD1);
		mongoOperation.save(userD1);
		User userE1 = mongoOperation.findOne(
                         new Query(Criteria.where("age").is(100)), User.class);
		System.out.println(userE1);
	}

}
```
