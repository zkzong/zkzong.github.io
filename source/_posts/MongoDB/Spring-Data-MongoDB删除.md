在Spring Data MongoDB中，可以使用`remove()`和`findAndRemove()`删除文档。

+ 1. remove() - 删除一个或多个文档。
+ 2. findAndRemove() - 删除单个文档，并返回删除的文档。

***常见错误：***
*不要使用`findAndRemove()`执行批量删除，因为只有符合条件的第一个文档被删除了。*

## 1. 删除文档实例

```
import java.util.ArrayList;
import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;

import com.mkyong.config.SpringMongoConfig;
import com.mkyong.model.User;

/**
 * Delete example
 *
 * @author mkyong
 *
 */

public class DeleteApp {

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
		User user5 = new User("1005", "elephant", 50);
		User user6 = new User("1006", "frog", 60);
		users.add(user1);
		users.add(user2);
		users.add(user3);
		users.add(user4);
		users.add(user5);
		users.add(user6);
		mongoOperation.insert(users, User.class);

		Query query1 = new Query();
		query1.addCriteria(Criteria.where("name").exists(true)
			.orOperator(
                           Criteria.where("name").is("frog"),
                           Criteria.where("name").is("dog")
                        ));
		mongoOperation.remove(query1, User.class);

		Query query2 = new Query();
		query2.addCriteria(Criteria.where("name").is("bird"));
		User userTest2 = mongoOperation.findOne(query2, User.class);
		mongoOperation.remove(userTest2);

		// The first document that matches the query is returned and also
		// removed from the collection in the database.
		Query query3 = new Query();
		query3.addCriteria(Criteria.where("name").is("ant"));
		User userTest3 = mongoOperation.findAndRemove(query3, User.class);
		System.out.println("Deleted document : " + userTest3);

		// either cat or elephant is deleted only,
                // common mistake, don't use for batch delete.
		/*
		  Query query4 = new Query();
                  query4.addCriteria(Criteria.where("name") .exists(true)
		       .orOperator(
                             Criteria.where("name").is("cat"),
		             Criteria.where("name").is("elephant")
                        )
                  );
		  mongoOperation.findAndRemove(query4, User.class);
		  System.out.println("Deleted document : " + userTest4);
		 */

		System.out.println("\nAll users : ");
		List<User> allUsers = mongoOperation.findAll(User.class);
		for (User user : allUsers) {
			System.out.println(user);
		}

		mongoOperation.dropCollection(User.class);

	}

}
```
输出：
```
Deleted document : User [id=5162e0153004c3cb0a907370, ic=1001, name=ant, age=10]

All users :
User [id=5162e0153004c3cb0a907372, ic=1003, name=cat, age=30]
User [id=5162e0153004c3cb0a907374, ic=1005, name=elephant, age=50]
```
