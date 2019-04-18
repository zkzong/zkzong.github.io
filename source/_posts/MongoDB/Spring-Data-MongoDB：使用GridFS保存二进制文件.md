在MongoDB中，可以使用GridFS保存二进制文件。本文介绍如何使用`GridFsTemplate`保存和读取图片文件。

## 1. GridFS - 保存（使用Spring注解方式）

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.config.AbstractMongoConfiguration;
import org.springframework.data.mongodb.gridfs.GridFsTemplate;

import com.mongodb.Mongo;
import com.mongodb.MongoClient;

/**
 * Spring MongoDB configuration file
 *
 */
@Configuration
public class SpringMongoConfig extends AbstractMongoConfiguration{

	@Bean
	public GridFsTemplate gridFsTemplate() throws Exception {
		return new GridFsTemplate(mongoDbFactory(), mappingMongoConverter());
	}

	@Override
	protected String getDatabaseName() {
		return "yourdb";
	}

	@Override
	@Bean
	public Mongo mongo() throws Exception {
		return new MongoClient("127.0.0.1");
	}

}
```

```
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.data.mongodb.gridfs.GridFsOperations;

import com.mkyong.config.SpringMongoConfig;
import com.mongodb.BasicDBObject;
import com.mongodb.DBObject;

/**
 * GridFs example
 *
 * @author zong
 *
 */

public class GridFsAppStore {

    public static void main(String[] args) {

	ApplicationContext ctx =
                     new AnnotationConfigApplicationContext(SpringMongoConfig.class);
	GridFsOperations gridOperations =
                      (GridFsOperations) ctx.getBean("gridFsTemplate");

	DBObject metaData = new BasicDBObject();
	metaData.put("extra1", "anything 1");
	metaData.put("extra2", "anything 2");

	InputStream inputStream = null;
	try {
		inputStream = new FileInputStream("/Users/mkyong/Downloads/testing.png");
		gridOperations.store(inputStream, "testing.png", "image/png", metaData);

	} catch (FileNotFoundException e) {
		e.printStackTrace();
	} finally {
		if (inputStream != null) {
			try {
				inputStream.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

		System.out.println("Done");
    }

}
```

使用MongoDB控制台查看保存的文件：
```
> show dbs
yourdb	0.203125GB

> use yourdb
switched to db yourdb

> show collections
fs.chunks
fs.files
system.indexes

> db.fs.files.find()
{ "_id" : ObjectId("51641c5630045c9b3db35afc"), "chunkSize" : NumberLong(262144),
"length" : NumberLong(4238), "md5" : "9969527cd95a5a573f15e953f0036800", "filename" : "testing.png",
"contentType" : "image/png", "uploadDate" : ISODate("2013-04-09T13:49:10.104Z"),
"aliases" : null, "metadata" : { "extra1" : "anything 1", "extra2" : "anything 2" } }
>

> db.fs.chunks.find()
{ "_id" : ObjectId("51641c5630045c9b3db35afd"),
"files_id" : ObjectId("51641c5630045c9b3db35afc"), "n" : 0,
"data" : BinData(0,"/9j/4AAQSkZJRgABAgAAZ......EQH/9k=") }
```
图片信息保存在`fs.files`，图片的二进制流保存在`fs.chunks`，通过`_id`和`files_id`关联。

## 2. GridFS - 读取（使用xml方式）

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:mongo="http://www.springframework.org/schema/data/mongo"
	xsi:schemaLocation="http://www.springframework.org/schema/data/mongo
       http://www.springframework.org/schema/data/mongo/spring-mongo.xsd
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

	<mongo:db-factory id="mongoDbFactory" dbname="yourdb" />
	<mongo:mapping-converter id="converter" />

	<bean name="gridFsTemplate"
		class="org.springframework.data.mongodb.gridfs.GridFsTemplate">
		<constructor-arg ref="mongoDbFactory" />
		<constructor-arg ref="converter" />
	</bean>

</beans>
```

```
import java.io.IOException;
import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.gridfs.GridFsOperations;

import com.mongodb.BasicDBObject;
import com.mongodb.DBObject;
import com.mongodb.gridfs.GridFSDBFile;

/**
 * GridFs example
 *
 * @author zong
 *
 */

public class GridFsAppRead {

    public static void main(String[] args) {

	ApplicationContext ctx =
              new GenericXmlApplicationContext("SpringConfig.xml");
	GridFsOperations gridOperations =
              (GridFsOperations) ctx.getBean("gridFsTemplate");

	List<GridFSDBFile> result = gridOperations.find(
               new Query().addCriteria(Criteria.where("filename").is("testing.png")));

	for (GridFSDBFile file : result) {
		try {
			System.out.println(file.getFilename());
			System.out.println(file.getContentType());

			//save as another image
			file.writeTo("/Users/mkyong/Downloads/new-testing.png");
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	System.out.println("Done");

    }
}
```
