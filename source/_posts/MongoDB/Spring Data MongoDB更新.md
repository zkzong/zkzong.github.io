---
title: Spring Data MongoDB更新
date: 2019-04-21
categories: MongoDB
---

在Spring Data MongoDB中，可以使用如下方法更新文档：
1. save - 如果`_id`存在则更新，否则插入。更新整个文档。
2. updateFirst - 更新查询出的第一个文档。
3. updateMulti - 更新查询出的所有文档。
4. upsert - 如果没有查询出文档，则会创建一个新文档。
5. findAndModify - 和updateMulti相同，但是它有一个额外的选项可以返回更新前或更新后的文档。

## 1. saveOrUpdate - 例1

假设下面的json数据插入到MongoDB。
```
{
	"_id" : ObjectId("id"),
	"ic" : "1001",
	"name" : "appleA",
	"age" : 20,
	"createdDate" : ISODate("2013-04-06T23:17:35.530Z")
}
```

查询文档，并使用`save()`方法修改和更新。
```java
Query query = new Query();
query.addCriteria(Criteria.where("name").is("appleA"));

User userTest1 = mongoOperation.findOne(query, User.class);

System.out.println("userTest1 - " + userTest1);

//modify and update with save()
userTest1.setAge(99);
mongoOperation.save(userTest1);

//get the updated object again
User userTest1_1 = mongoOperation.findOne(query, User.class);

System.out.println("userTest1_1 - " + userTest1_1);
```
输出：
```
userTest1 - User [id=id, ic=1001, name=appleA, age=20, createdDate=Sat Apr 06 23:17:35 MYT 2013]
userTest1_1 - User [id=id, ic=1001, name=appleA, age=99, createdDate=Sat Apr 06 23:17:35 MYT 2013]
```

## 1. saveOrUpdate - 例2

这是一个失败的例子，请仔细阅读，这是一个相当普遍的错误。

假设下面的json数据插入到MongoDB。
```
{
	"_id" : ObjectId("id"),
	"ic" : "1002",
	"name" : "appleB",
	"age" : 20,
	"createdDate" : ISODate("2013-04-06T15:22:34.530Z")
}
```
使用`Query`获取只包含`name`字段的文档，返回的User对象的age、ic和createDate字段都为null，如果要修改age字段并更新，它会覆盖所有值而不是更新age字段。
```java
Query query = new Query();
query.addCriteria(Criteria.where("name").is("appleB"));
query.fields().include("name");

User userTest2 = mongoOperation.findOne(query, User.class);
System.out.println("userTest2 - " + userTest2);

userTest2.setAge(99);

mongoOperation.save(userTest2);

// ooppss, you just override everything, it caused ic=null and
// createdDate=null

Query query1 = new Query();
query1.addCriteria(Criteria.where("name").is("appleB"));

User userTest2_1 = mongoOperation.findOne(query1, User.class);
System.out.println("userTest2_1 - " + userTest2_1);
```
输出：
```
userTest2 - User [id=51603dba3004d7fffc202391, ic=null, name=appleB, age=0, createdDate=null]
userTest2_1 - User [id=51603dba3004d7fffc202391, ic=null, name=appleB, age=99, createdDate=null]
```
`save()`方法之后，age字段被正确更新了，但是ic和createdDate字段都是null，整个user对象都被更新了。为了只更新某个字段而不是整个对象，可以使用`updateFirst()`或者`updateMulti()`，而不是`save()`。

## 3. updateFirst

更新符合查询条件的第一个文档。本例中，只有`age`字段被更新。
```
{
	"_id" : ObjectId("id"),
	"ic" : "1003",
	"name" : "appleC",
	"age" : 20,
	"createdDate" : ISODate("2013-04-06T23:22:34.530Z")
}
```
```
//returns only 'name' field
Query query = new Query();
query.addCriteria(Criteria.where("name").is("appleC"));
query.fields().include("name");

User userTest3 = mongoOperation.findOne(query, User.class);
System.out.println("userTest3 - " + userTest3);

Update update = new Update();
update.set("age", 100);

mongoOperation.updateFirst(query, update, User.class);

//returns everything
Query query1 = new Query();
query1.addCriteria(Criteria.where("name").is("appleC"));

User userTest3_1 = mongoOperation.findOne(query1, User.class);
System.out.println("userTest3_1 - " + userTest3_1);
```
输出：
```
userTest3 - User [id=id, ic=null, name=appleC, age=0, createdDate=null]
userTest3_1 - User [id=id, ic=1003, name=appleC, age=100, createdDate=Sat Apr 06 23:22:34 MYT 2013]
```

## 4. updateMulti

更新符合查询条件的所有文档。
```
{
	"_id" : ObjectId("id"),
	"ic" : "1004",
	"name" : "appleD",
	"age" : 20,
	"createdDate" : ISODate("2013-04-06T15:22:34.530Z")
}
{
	"_id" : ObjectId("id"),
	"ic" : "1005",
	"name" : "appleE",
	"age" : 20,
	"createdDate" : ISODate("2013-04-06T15:22:34.530Z")
}
```
```
//show the use of $or operator
Query query = new Query();
query.addCriteria(Criteria
		.where("name").exists(true)
		.orOperator(Criteria.where("name").is("appleD"),
				Criteria.where("name").is("appleE")));
Update update = new Update();

//update age to 11
update.set("age", 11);

//remove the createdDate field
update.unset("createdDate");

// if use updateFirst, it will update 1004 only.
// mongoOperation.updateFirst(query4, update4, User.class);

// update all matched, both 1004 and 1005
mongoOperation.updateMulti(query, update, User.class);

System.out.println(query.toString());

List<User> usersTest4 = mongoOperation.find(query4, User.class);

for (User userTest4 : usersTest4) {
	System.out.println("userTest4 - " + userTest4);
}
```
输出：
```
Query: { "name" : { "$exists" : true} ,
	"$or" : [ { "name" : "appleD"} , { "name" : "appleE"}]}, Fields: null, Sort: null

userTest4 - User [id=id, ic=1004, name=appleD, age=11, createdDate=null]
userTest4 - User [id=id, ic=1005, name=appleE, age=11, createdDate=null]
```

## 5. upsert

如果查询到符合的文档就更新，如果没有就根据查询条件创建一个新的文档，就像`findAndModifyElseCreate()`。
```
{
	//no data
}
```
```java
//search a document that doesn't exist
Query query = new Query();
query.addCriteria(Criteria.where("name").is("appleZ"));

Update update = new Update();
update.set("age", 21);

mongoOperation.upsert(query, update, User.class);

User userTest5 = mongoOperation.findOne(query, User.class);
System.out.println("userTest5 - " + userTest5);
```
输出：
```
userTest5 - User [id=id, ic=null, name=appleZ, age=21, createdDate=null]
```

## 6. findAndModify

使用一个操作查询、修改并获得更新后的对象。
```
{
	"_id" : ObjectId("id"),
	"ic" : "1006",
	"name" : "appleF",
	"age" : 20,
	"createdDate" : ISODate("2013-04-07T13:11:34.530Z")
}
```
```
Query query6 = new Query();
query6.addCriteria(Criteria.where("name").is("appleF"));

Update update6 = new Update();
update6.set("age", 101);
update6.set("ic", 1111);

//FindAndModifyOptions().returnNew(true) = newly updated document
//FindAndModifyOptions().returnNew(false) = old document (not update yet)
User userTest6 = mongoOperation.findAndModify(
		query6, update6,
		new FindAndModifyOptions().returnNew(true), User.class);
System.out.println("userTest6 - " + userTest6);
```
输出：
```
userTest6 - User [id=id, ic=1111, name=appleF, age=101, createdDate=Sun Apr 07 13:11:34 MYT 2013]
```

## 7. 完整实例

```
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.data.mongodb.core.FindAndModifyOptions;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;

import com.mkyong.config.SpringMongoConfig;
import com.mkyong.model.User;

public class UpdateApp {

	public static void main(String[] args) {
		// For Annotation
		ApplicationContext ctx =
			new AnnotationConfigApplicationContext(SpringMongoConfig.class);
		MongoOperations mongoOperation =
			(MongoOperations) ctx.getBean("mongoTemplate");

		// insert 6 users for testing
		List<User> users = new ArrayList<User>();

		User user1 = new User("1001", "appleA", 20, new Date());
		User user2 = new User("1002", "appleB", 20, new Date());
		User user3 = new User("1003", "appleC", 20, new Date());
		User user4 = new User("1004", "appleD", 20, new Date());
		User user5 = new User("1005", "appleE", 20, new Date());
		User user6 = new User("1006", "appleF", 20, new Date());
		users.add(user1);
		users.add(user2);
		users.add(user3);
		users.add(user4);
		users.add(user5);
		users.add(user6);
		mongoOperation.insert(users, User.class);

		// Case 1 ... find and update
		System.out.println("Case 1");

		Query query1 = new Query();
		query1.addCriteria(Criteria.where("name").is("appleA"));

		User userTest1 = mongoOperation.findOne(query1, User.class);

		System.out.println("userTest1 - " + userTest1);

		userTest1.setAge(99);
		mongoOperation.save(userTest1);

		User userTest1_1 = mongoOperation.findOne(query1, User.class);

		System.out.println("userTest1_1 - " + userTest1_1);

		// Case 2 ... select single field only
		System.out.println("\nCase 2");

		Query query2 = new Query();
		query2.addCriteria(Criteria.where("name").is("appleB"));
		query2.fields().include("name");

		User userTest2 = mongoOperation.findOne(query2, User.class);
		System.out.println("userTest2 - " + userTest2);

		userTest2.setAge(99);

		mongoOperation.save(userTest2);

		// ooppss, you just override everything, it caused ic=null and
		// createdDate=null

		Query query2_1 = new Query();
		query2_1.addCriteria(Criteria.where("name").is("appleB"));

		User userTest2_1 = mongoOperation.findOne(query2_1, User.class);
		System.out.println("userTest2_1 - " + userTest2_1);

		System.out.println("\nCase 3");
		Query query3 = new Query();
		query3.addCriteria(Criteria.where("name").is("appleC"));
		query3.fields().include("name");

		User userTest3 = mongoOperation.findOne(query3, User.class);
		System.out.println("userTest3 - " + userTest3);

		Update update3 = new Update();
		update3.set("age", 100);

		mongoOperation.updateFirst(query3, update3, User.class);

		Query query3_1 = new Query();
		query3_1.addCriteria(Criteria.where("name").is("appleC"));

		User userTest3_1 = mongoOperation.findOne(query3_1, User.class);
		System.out.println("userTest3_1 - " + userTest3_1);

		System.out.println("\nCase 4");
		Query query4 = new Query();
		query4.addCriteria(Criteria
				.where("name")
				.exists(true)
				.orOperator(Criteria.where("name").is("appleD"),
						Criteria.where("name").is("appleE")));
		Update update4 = new Update();
		update4.set("age", 11);
		update4.unset("createdDate");

		// update 1004 only.
		// mongoOperation.updateFirst(query4, update4, User.class);

		// update all matched
		mongoOperation.updateMulti(query4, update4, User.class);

		System.out.println(query4.toString());

		List<User> usersTest4 = mongoOperation.find(query4, User.class);

		for (User userTest4 : usersTest4) {
			System.out.println("userTest4 - " + userTest4);
		}

		System.out.println("\nCase 5");
		Query query5 = new Query();
		query5.addCriteria(Criteria.where("name").is("appleZ"));

		Update update5 = new Update();
		update5.set("age", 21);

		mongoOperation.upsert(query5, update5, User.class);

		User userTest5 = mongoOperation.findOne(query5, User.class);
		System.out.println("userTest5 - " + userTest5);

		System.out.println("\nCase 6");
		Query query6 = new Query();
		query6.addCriteria(Criteria.where("name").is("appleF"));

		Update update6 = new Update();
		update6.set("age", 101);
		update6.set("ic", 1111);

		User userTest6 = mongoOperation.findAndModify(query6, update6,
				new FindAndModifyOptions().returnNew(true), User.class);
		System.out.println("userTest6 - " + userTest6);

		mongoOperation.dropCollection(User.class);

	}

}
```
输出：
```
Case 1
userTest1 - User [id=id, ic=1001, name=appleA, age=20, createdDate=Sun Apr 07 13:22:48 MYT 2013]
userTest1_1 - User [id=id, ic=1001, name=appleA, age=99, createdDate=Sun Apr 07 13:22:48 MYT 2013]

Case 2
userTest2 - User [id=id, ic=null, name=appleB, age=0, createdDate=null]
userTest2_1 - User [id=id, ic=null, name=appleB, age=99, createdDate=null]

Case 3
userTest3 - User [id=id, ic=null, name=appleC, age=0, createdDate=null]
userTest3_1 - User [id=id, ic=1003, name=appleC, age=100, createdDate=Sun Apr 07 13:22:48 MYT 2013]

Case 4
Query: { "name" : { "$exists" : true} , "$or" : [ { "name" : "appleD"} , { "name" : "appleE"}]}, Fields: null, Sort: null
userTest4 - User [id=id, ic=1004, name=appleD, age=11, createdDate=null]
userTest4 - User [id=id, ic=1005, name=appleE, age=11, createdDate=null]

Case 5
userTest5 - User [id=id, ic=null, name=appleZ, age=21, createdDate=null]

Case 6
userTest6 - User [id=id, ic=1006, name=appleF, age=20, createdDate=Sun Apr 07 13:22:48 MYT 2013]
```
