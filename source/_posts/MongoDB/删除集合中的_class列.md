默认情况下，Spring Data的`MappingMongoConverter`为MongoDb中的每个对象添加了一个额外的`_class`列。例如：
```
public class User {

	String username;
	String password;

	//...getters and setters
}
```
保存：
```
MongoOperations mongoOperation = (MongoOperations)ctx.getBean("mongoTemplate");
User user = new User("mkyong", "password123");
mongoOperation.save(user, "users");
```
结果：
```
> db.users.find()
{
	"_class" : "com.mkyong.user.User",
	"_id" : ObjectId("5050aef830041f24ff2bd16e"),
	"password" : "new password", "username" : "mkyong"
}
```
Spring Data创建了额外的`_class`列。为了删除额外的`_class`列，重写`MappingMongoConverter`，传入`new DefaultMongoTypeMapper(null)`。

下面介绍了两种删除`_class`的方法：注解和xml。

## 1. 注解

```
@Configuration
public class SpringMongoConfig {

  @Bean
  public MongoDbFactory mongoDbFactory() throws Exception {
	return new SimpleMongoDbFactory(new Mongo(), "database");
  }

  @Bean
  public MongoTemplate mongoTemplate() throws Exception {

	//remove _class
	MappingMongoConverter converter =
		new MappingMongoConverter(mongoDbFactory(), new MongoMappingContext());
	converter.setTypeMapper(new DefaultMongoTypeMapper(null));

	MongoTemplate mongoTemplate = new MongoTemplate(mongoDbFactory(), converter);

	return mongoTemplate;
  }

}
```

## 2. XML

```
<mongo:mongo host="localhost" port="27017" />
<mongo:db-factory dbname="database" />

 <bean id="mappingContext"
	class="org.springframework.data.mongodb.core.mapping.MongoMappingContext" />

 <bean id="defaultMongoTypeMapper"
	class="org.springframework.data.mongodb.core.convert.DefaultMongoTypeMapper">
	<constructor-arg name="typeKey"><null/></constructor-arg>
 </bean>

 <bean id="mappingMongoConverter"
	class="org.springframework.data.mongodb.core.convert.MappingMongoConverter">
	<constructor-arg name="mongoDbFactory" ref="mongoDbFactory" />
	<constructor-arg name="mappingContext" ref="mappingContext" />
	<property name="typeMapper" ref="defaultMongoTypeMapper" />
 </bean>

 <bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
	<constructor-arg name="mongoDbFactory" ref="mongoDbFactory" />
	<constructor-arg name="mongoConverter" ref="mappingMongoConverter" />
 </bean>
```

## 3. Spring Boot

```
@Bean
public MongoTemplate mongoTemplate(MongoDbFactory mongoDbFactory, MongoMappingContext context) {

    MappingMongoConverter converter = new MappingMongoConverter(new DefaultDbRefResolver(mongoDbFactory), context);
    converter.setTypeMapper(new DefaultMongoTypeMapper(null));

    MongoTemplate mongoTemplate = new MongoTemplate(mongoDbFactory, converter);

    return mongoTemplate;

}
```

## 4. 测试

`_class`已经被删除了。

```
> db.users.find()
{
	"_id" : ObjectId("random code"),
	"password" : "new password", "username" : "mkyong"
}
```
