---
title: Java MongoDB基本操作
date: 2019-04-21
categories: MongoDB
---

本文介绍如何使用Java操作MongoDB，如创建连接数据库、集合和文档，保存、更新、删除和查询文档。

## 1. 引入依赖

```
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.5.0</version>
</dependency>
```

## 2. Mongo连接

使用`MongoClient`连接到数据库。

```
MongoClient mongo = new MongoClient("localhost", 27017);
```

如果是安全模式，需要认证。

```
MongoClient mongoClient = new MongoClient();
DB db = mongoClient.getDB("database name");
boolean auth = db.authenticate("username", "password".toCharArray());
```

## 3. MongoDB数据库

获取数据库：

```
MongoDatabase database = mongoClient.getDatabase("t");
```

显示所有数据库：

```
MongoIterable<String> dbs = mongoClient.listDatabaseNames();
for (String db : dbs) {
    System.out.println(db);
}
```

## 4. 集合

获取集合：
```
MongoDatabase mongoDatabase = mongoClient.getDatabase("test");
MongoCollection<Document> collection = mongoDatabase.getCollection("coll");
```
显示所有集合
```
MongoIterable<String> collections = mongoDatabase.listCollectionNames();
for (String c : collections) {
    System.out.println(c);
}
```

## 5. 保存

```
DBCollection table = db.getCollection("user");
BasicDBObject document = new BasicDBObject();
document.put("name", "mkyong");
document.put("age", 30);
document.put("createdDate", new Date());
table.insert(document);
```

## 6. 更新

```
DBCollection table = db.getCollection("user");

BasicDBObject query = new BasicDBObject();
query.put("name", "mkyong");

BasicDBObject newDocument = new BasicDBObject();
newDocument.put("name", "mkyong-updated");

BasicDBObject updateObj = new BasicDBObject();
updateObj.put("$set", newDocument);

table.update(query, updateObj);
```

## 7. 查询

```
DBCollection table = db.getCollection("user");

BasicDBObject searchQuery = new BasicDBObject();
searchQuery.put("name", "mkyong");

DBCursor cursor = table.find(searchQuery);

while (cursor.hasNext()) {
	System.out.println(cursor.next());
}
```

## 8. 删除

```
DBCollection table = db.getCollection("user");

BasicDBObject searchQuery = new BasicDBObject();
searchQuery.put("name", "mkyong");

table.remove(searchQuery);
```

## 完整例子

```
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;
import com.mongodb.MongoClient;
import com.mongodb.MongoException;
import com.mongodb.client.MongoDatabase;

import java.util.Date;

public class HelloWorld {
    public static void main(String[] args) {
        try {

            /**** Connect to MongoDB ****/
            // Since 2.10.0, uses MongoClient
            MongoClient mongo = new MongoClient("localhost", 27017);

            /**** Get database ****/
            // if database doesn't exists, MongoDB will create it for you
            DB db = mongo.getDB("testdb");
            MongoDatabase mongoDatabase = mongo.getDatabase("testdb");

            /**** Get collection / table from 'testdb' ****/
            // if collection doesn't exists, MongoDB will create it for you
            DBCollection table = db.getCollection("user");

            /**** Insert ****/
            // create a document to store key and value
            BasicDBObject document = new BasicDBObject();
            document.put("name", "mkyong");
            document.put("age", 30);
            document.put("createdDate", new Date());
            table.insert(document);

            /**** Find and display ****/
            BasicDBObject searchQuery = new BasicDBObject();
            searchQuery.put("name", "mkyong");

            DBCursor cursor = table.find(searchQuery);

            while (cursor.hasNext()) {
                System.out.println(cursor.next());
            }

            /**** Update ****/
            // search document where name="mkyong" and update it with new values
            BasicDBObject query = new BasicDBObject();
            query.put("name", "mkyong");

            BasicDBObject newDocument = new BasicDBObject();
            newDocument.put("name", "mkyong-updated");

            BasicDBObject updateObj = new BasicDBObject();
            updateObj.put("$set", newDocument);

            table.update(query, updateObj);

            /**** Find and display ****/
            BasicDBObject searchQuery2
                    = new BasicDBObject().append("name", "mkyong-updated");

            DBCursor cursor2 = table.find(searchQuery2);

            while (cursor2.hasNext()) {
                System.out.println(cursor2.next());
            }

            /**** Done ****/
            System.out.println("Done");

        } catch (MongoException e) {
            e.printStackTrace();
        }
    }
}
```
