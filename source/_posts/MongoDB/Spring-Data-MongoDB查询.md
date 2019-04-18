本文介绍一下使用Query、Criteria和其他常见操作查询文档的例子。

**测试数据**
```
> db.users.find()
{ "_id" : ObjectId("id"), "ic" : "1001", "name" : "ant", "age" : 10 }
{ "_id" : ObjectId("id"), "ic" : "1002", "name" : "bird", "age" : 20 }
{ "_id" : ObjectId("id"), "ic" : "1003", "name" : "cat", "age" : 30 }
{ "_id" : ObjectId("id"), "ic" : "1004", "name" : "dog", "age" : 40 }
{ "_id" : ObjectId("id"), "ic" : "1005", "name" : "elephant", "age" : 50 }
{ "_id" : ObjectId("id"), "ic" : "1006", "name" : "frog", "age" : 60 }
```

## 1. BasicQuery

如果数据MongoDB控制台的`find()`命令，只需要把原生的查询放到BasicQuery就可以了。
```
BasicQuery query1 = new BasicQuery("{ age : { $lt : 40 }, name : 'cat' }");
User userTest1 = mongoOperation.findOne(query1, User.class);

System.out.println("query1 - " + query1.toString());
System.out.println("userTest1 - " + userTest1);
```
输出：
```
query1 - Query: { "age" : { "$lt" : 40} , "name" : "cat"}, Fields: null, Sort: { }
userTest1 - User [id=id, ic=1003, name=cat, age=30]
```

## 2. findOne

findOne返回符合查询条件的一个wendang文档，如果有多个查询条件，可以使用`Criteria.and()`方法。
```
Query query2 = new Query();
query2.addCriteria(Criteria.where("name").is("dog").and("age").is(40));

User userTest2 = mongoOperation.findOne(query2, User.class);
System.out.println("query2 - " + query2.toString());
System.out.println("userTest2 - " + userTest2);
```
输出：
```
query2 - Query: { "name" : "dog" , "age" : 40}, Fields: null, Sort: null
userTest2 - User [id=id, ic=1004, name=dog, age=40]
```

## 3. find和$in

查询并返回符合查询的文档列表。本例也展示了`$in`的用法。
```
List<Integer> listOfAge = new ArrayList<Integer>();
listOfAge.add(10);
listOfAge.add(30);
listOfAge.add(40);

Query query3 = new Query();
query3.addCriteria(Criteria.where("age").in(listOfAge));

List<User> userTest3 = mongoOperation.find(query3, User.class);
System.out.println("query3 - " + query3.toString());

for (User user : userTest3) {
	System.out.println("userTest3 - " + user);
}
```
输出：
```
query3 - Query: { "age" : { "$in" : [ 10 , 30 , 40]}}, Fields: null, Sort: null
userTest3 - User [id=id, ic=1001, name=ant, age=10]
userTest3 - User [id=id, ic=1003, name=cat, age=30]
userTest3 - User [id=id, ic=1004, name=dog, age=40]
```

## find，$gt，$lt，$and

查询并返回符合查询的文档列表。本例也展示了`$gt`、`$lt`和`$and`的用法。
```
Query query4 = new Query();
query4.addCriteria(Criteria.where("age").lt(40).and("age").gt(10));

List<User> userTest4 = mongoOperation.find(query4, User.class);
System.out.println("query4 - " + query4.toString());

for (User user : userTest4) {
	System.out.println("userTest4 - " + user);
}
```
上面例子报错了：
```
Due to limitations of the com.mongodb.BasicDBObject, you can't add a second 'age' expression
specified as 'age : { "$gt" : 10}'. Criteria already contains 'age : { "$lt" : 40}'.
```
提示说不能在同一个字段上使用`Criteria.and()`，可以使用`Criteria.andOperator()`修复改问题。
```
Query query4 = new Query();
query4.addCriteria(
	Criteria.where("age").exists(true)
	.andOperator(
		Criteria.where("age").gt(10),
                Criteria.where("age").lt(40)
	)
);

List<User> userTest4 = mongoOperation.find(query4, User.class);
System.out.println("query4 - " + query4.toString());

for (User user : userTest4) {
	System.out.println("userTest4 - " + user);
}
```
输出：
```
query4 - Query: { "age" : { "$lt" : 40} , "$and" : [ { "age" : { "$gt" : 10}}]}, Fields: null, Sort: null
userTest4 - User [id=51627a0a3004cc5c0af72964, ic=1002, name=bird, age=20]
userTest4 - User [id=51627a0a3004cc5c0af72965, ic=1003, name=cat, age=30]
```

## 5. find并排序

```
Query query5 = new Query();
query5.addCriteria(Criteria.where("age").gte(30));
query5.with(new Sort(Sort.Direction.DESC, "age"));

List<User> userTest5 = mongoOperation.find(query5, User.class);
System.out.println("query5 - " + query5.toString());

for (User user : userTest5) {
	System.out.println("userTest5 - " + user);
}
```
输出：
```
query5 - Query: { "age" : { "$gte" : 30}}, Fields: null, Sort: { "age" : -1}
userTest5 - User [id=id, ic=1006, name=frog, age=60]
userTest5 - User [id=id, ic=1005, name=elephant, age=50]
userTest5 - User [id=id, ic=1004, name=dog, age=40]
userTest5 - User [id=id, ic=1003, name=cat, age=30]
```

## 6. find和$regex

使用正则表达式查找。

```
Query query6 = new Query();
query6.addCriteria(Criteria.where("name").regex("D.*G", "i"));

List<User> userTest6 = mongoOperation.find(query6, User.class);
System.out.println("query6 - " + query6.toString());

for (User user : userTest6) {
	System.out.println("userTest6 - " + user);
}
```
输出：
```
query6 - Query: { "name" : { "$regex" : "D.*G" , "$options" : "i"}}, Fields: null, Sort: null
userTest6 - User [id=id, ic=1004, name=dog, age=40]
```

## 7. 完整实例

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

	//getter, setter and constructor methods

}
```

```
import java.util.ArrayList;
import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.data.domain.Sort;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.query.BasicQuery;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;

import com.mkyong.config.SpringMongoConfig;
import com.mkyong.model.User;

/**
 * Query example
 *
 * @author mkyong
 *
 */
public class QueryApp {

	public static void main(String[] args) {

		ApplicationContext ctx =
                         new AnnotationConfigApplicationContext(SpringMongoConfig.class);
		MongoOperations mongoOperation =
                         (MongoOperations) ctx.getBean("mongoTemplate");

		// insert 6 users for testing
		List<User> users = new ArrayList<User>();

		User user1 = new User("1001", "ant", 10);
		User user2 = new User("1002", "bird", 20);
		User user3 = new User("1003", "cat", 30);
		User user4 = new User("1004", "dog", 40);
		User user5 = new User("1005", "elephant",50);
		User user6 = new User("1006", "frog", 60);
		users.add(user1);
		users.add(user2);
		users.add(user3);
		users.add(user4);
		users.add(user5);
		users.add(user6);
		mongoOperation.insert(users, User.class);

		System.out.println("Case 1 - find with BasicQuery example");

		BasicQuery query1 = new BasicQuery("{ age : { $lt : 40 }, name : 'cat' }");
		User userTest1 = mongoOperation.findOne(query1, User.class);

		System.out.println("query1 - " + query1.toString());
		System.out.println("userTest1 - " + userTest1);

		System.out.println("\nCase 2 - find example");

		Query query2 = new Query();
		query2.addCriteria(Criteria.where("name").is("dog").and("age").is(40));

		User userTest2 = mongoOperation.findOne(query2, User.class);
		System.out.println("query2 - " + query2.toString());
		System.out.println("userTest2 - " + userTest2);

		System.out.println("\nCase 3 - find list $inc example");

		List<Integer> listOfAge = new ArrayList<Integer>();
		listOfAge.add(10);
		listOfAge.add(30);
		listOfAge.add(40);

		Query query3 = new Query();
		query3.addCriteria(Criteria.where("age").in(listOfAge));

		List<User> userTest3 = mongoOperation.find(query3, User.class);
		System.out.println("query3 - " + query3.toString());

		for (User user : userTest3) {
			System.out.println("userTest3 - " + user);
		}

		System.out.println("\nCase 4 - find list $and $lt, $gt example");

		Query query4 = new Query();

		// it hits error
		// query4.addCriteria(Criteria.where("age").lt(40).and("age").gt(10));

		query4.addCriteria(
                   Criteria.where("age").exists(true).andOperator(
		         Criteria.where("age").gt(10),
                         Criteria.where("age").lt(40)
	            )
                );

		List<User> userTest4 = mongoOperation.find(query4, User.class);
		System.out.println("query4 - " + query4.toString());

		for (User user : userTest4) {
			System.out.println("userTest4 - " + user);
		}

		System.out.println("\nCase 5 - find list and sorting example");
		Query query5 = new Query();
		query5.addCriteria(Criteria.where("age").gte(30));
		query5.with(new Sort(Sort.Direction.DESC, "age"));

		List<User> userTest5 = mongoOperation.find(query5, User.class);
		System.out.println("query5 - " + query5.toString());

		for (User user : userTest5) {
			System.out.println("userTest5 - " + user);
		}

		System.out.println("\nCase 6 - find by regex example");
		Query query6 = new Query();
		query6.addCriteria(Criteria.where("name").regex("D.*G", "i"));

		List<User> userTest6 = mongoOperation.find(query6, User.class);
		System.out.println("query6 - " + query6.toString());

		for (User user : userTest6) {
			System.out.println("userTest6 - " + user);
		}

		mongoOperation.dropCollection(User.class);

	}

}
```
输出：
```
Case 1 - find with BasicQuery example
query1 - Query: { "age" : { "$lt" : 40} , "name" : "cat"}, Fields: null, Sort: { }
userTest1 - User [id=id, ic=1003, name=cat, age=30]

Case 2 - find example
query2 - Query: { "name" : "dog" , "age" : 40}, Fields: null, Sort: null
userTest2 - User [id=id, ic=1004, name=dog, age=40]

Case 3 - find list $inc example
query3 - Query: { "age" : { "$in" : [ 10 , 30 , 40]}}, Fields: null, Sort: null
userTest3 - User [id=id, ic=1001, name=ant, age=10]
userTest3 - User [id=id, ic=1003, name=cat, age=30]
userTest3 - User [id=id, ic=1004, name=dog, age=40]

Case 4 - find list $and $lt, $gt example
query4 - Query: { "age" : { "$lt" : 40} , "$and" : [ { "age" : { "$gt" : 10}}]}, Fields: null, Sort: null
userTest4 - User [id=id, ic=1002, name=bird, age=20]
userTest4 - User [id=id, ic=1003, name=cat, age=30]

Case 5 - find list and sorting example
query5 - Query: { "age" : { "$gte" : 30}}, Fields: null, Sort: { "age" : -1}
userTest5 - User [id=id, ic=1006, name=frog, age=60]
userTest5 - User [id=id, ic=1005, name=elephant, age=50]
userTest5 - User [id=id, ic=1004, name=dog, age=40]
userTest5 - User [id=id, ic=1003, name=cat, age=30]

Case 6 - find by regex example
query6 - Query: { "name" : { "$regex" : "D.*G" , "$options" : "i"}}, Fields: null, Sort: null
userTest6 - User [id=id, ic=1004, name=dog, age=40]
```
