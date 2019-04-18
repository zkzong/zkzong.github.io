本文介绍如何从集合中查询文档的通用方法。

## 测试数据

插入5条测试文档
```
{ "_id" : { "$oid" : "id"} , "number" : 1 , "name" : "mkyong-1"}
{ "_id" : { "$oid" : "id"} , "number" : 2 , "name" : "mkyong-2"}
{ "_id" : { "$oid" : "id"} , "number" : 3 , "name" : "mkyong-3"}
{ "_id" : { "$oid" : "id"} , "number" : 4 , "name" : "mkyong-4"}
{ "_id" : { "$oid" : "id"} , "number" : 5 , "name" : "mkyong-5"}
```

## 1. find()

### 1.1 获取第一条文档
```
Document document = collection.find().first();
System.out.println(document);
```
输出：
```
{ "_id" : { "$oid" : "id"} , "number" : 1 , "name" : "mkyong-1"}
```

### 1.2 获取所有文档
```
FindIterable<Document> documents = collection.find();
MongoCursor<Document> mongoCursor = documents.iterator();
while (mongoCursor.hasNext()) {
    System.out.println(mongoCursor.next());
}
```
输出：
```
{ "_id" : { "$oid" : "id"} , "number" : 1 , "name" : "mkyong-1"}
{ "_id" : { "$oid" : "id"} , "number" : 2 , "name" : "mkyong-2"}
{ "_id" : { "$oid" : "id"} , "number" : 3 , "name" : "mkyong-3"}
{ "_id" : { "$oid" : "id"} , "number" : 4 , "name" : "mkyong-4"}
{ "_id" : { "$oid" : "id"} , "number" : 5 , "name" : "mkyong-5"}
```

### 1.3 获取文档的单一字段
```
Document fields = new Document();
fields.put("name", 1);

FindIterable<Document> projection = collection.find().projection(fields);
MongoCursor<Document> iterator = projection.iterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```
输出：
```
{ "_id" : { "$oid" : "id"} , "name" : "mkyong-1"}
{ "_id" : { "$oid" : "id"} , "name" : "mkyong-2"}
{ "_id" : { "$oid" : "id"} , "name" : "mkyong-3"}
{ "_id" : { "$oid" : "id"} , "name" : "mkyong-4"}
{ "_id" : { "$oid" : "id"} , "name" : "mkyong-5"}
```

## 2. 使用find()对比查询

### 2.1 获取所有`number = 5`的文档
```
Document whereQuery = new Document();
whereQuery.put("number", 5);
FindIterable<Document> whereDocuments = collection.find(whereQuery);
MongoCursor<Document> whereIterator = whereDocuments.iterator();
while (whereIterator.hasNext()) {
    System.out.println(whereIterator.next());
}
```
输出：
```
{ "_id" : { "$oid" : "id"} , "number" : 5 , "name" : "mkyong-5"}
```

### 2.2 `$in` - 获取number在2、4、5中的文档
```
Document inQuery = new Document();
List<Integer> list = new ArrayList<Integer>();
list.add(2);
list.add(4);
list.add(5);
inQuery.put("number", new Document("$in", list));
FindIterable<Document> listDocuments = collection.find(inQuery);
MongoCursor<Document> listIterator = listDocuments.iterator();
while (listIterator.hasNext()) {
    System.out.println(listIterator.next());
}
```
输出：
```
{ "_id" : { "$oid" : "id"} , "number" : 2 , "name" : "mkyong-2"}
{ "_id" : { "$oid" : "id"} , "number" : 4 , "name" : "mkyong-4"}
{ "_id" : { "$oid" : "id"} , "number" : 5 , "name" : "mkyong-5"}
```

### 2.3 `$gt $lt` - 获取`5 > number > 2`的文档
```
Document gtQuery = new Document();
gtQuery.put("number", new Document("$gt", 2).append("$lt", 5));
FindIterable<Document> gtDocuments = collection.find(gtQuery);
MongoCursor<Document> gtIterator = gtDocuments.iterator();
while (gtIterator.hasNext()) {
    System.out.println(gtIterator.next());
}
```
输出：
```
{ "_id" : { "$oid" : "id"} , "number" : 3 , "name" : "mkyong-3"}
{ "_id" : { "$oid" : "id"} , "number" : 4 , "name" : "mkyong-4"}
```

### 2.4 `$ne` - 获取`number != 4`的文档
```
Document neQuery = new Document();
neQuery.put("number", new Document("$ne", 4));
FindIterable<Document> neDocuments = collection.find(neQuery);
MongoCursor<Document> neIterator = neDocuments.iterator();
while (neIterator.hasNext()) {
    System.out.println(neIterator.next());
}
```
输出：
```
{ "_id" : { "$oid" : "id"} , "number" : 1 , "name" : "mkyong-1"}
{ "_id" : { "$oid" : "id"} , "number" : 2 , "name" : "mkyong-2"}
{ "_id" : { "$oid" : "id"} , "number" : 3 , "name" : "mkyong-3"}
{ "_id" : { "$oid" : "id"} , "number" : 5 , "name" : "mkyong-5"}
```

## 3. 使用find()进行逻辑查询

### 3.1 `$and` - 获取`number = 2 and name = 'mkyong-2'`的文档

```
Document andQuery = new Document();

List<Document> obj = new ArrayList<Document>();
obj.add(new Document("number", 2));
obj.add(new Document("name", "mkyong-2"));
andQuery.put("$and", obj);

System.out.println(andQuery.toString());

FindIterable<Document> andDocuments = collection.find(andQuery);
MongoCursor<Document> andIterator = andDocuments.iterator();
while (andIterator.hasNext()) {
    System.out.println(andIterator.next());
}
```
输出：
```
{ "$and" : [ { "number" : 2} , { "name" : "mkyong-2"}]}

{ "_id" : { "$oid" : "id"} , "number" : 2 , "name" : "mkyong-2"}
```

## 4. 使用find()通过正则表达式查询

### 4.1 `$regex`

```
Document regexQuery = new Document();
regexQuery.put("name",
        new Document("$regex", "Mky.*-[1-3]")
                .append("$options", "i"));

System.out.println(regexQuery.toString());

FindIterable<Document> regexDocuments = collection.find(regexQuery);
MongoCursor<Document> regexIterator = regexDocuments.iterator();
while (regexIterator.hasNext()) {
    System.out.println(regexIterator.next());
}
```
输出：
```
{ "name" : { "$regex" : "Mky.*-[1-3]" , "$options" : "i"}}

{ "_id" : { "$oid" : "515ad59e3004c89329c7b259"} , "number" : 1 , "name" : "mkyong-1"}
{ "_id" : { "$oid" : "515ad59e3004c89329c7b25a"} , "number" : 2 , "name" : "mkyong-2"}
{ "_id" : { "$oid" : "515ad59e3004c89329c7b25b"} , "number" : 3 , "name" : "mkyong-3"}
```

## 5. 完整实例

```
import com.mongodb.MongoClient;
import com.mongodb.MongoException;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import java.util.ArrayList;
import java.util.Calendar;
import java.util.List;

public class FindDocument {
    public static void insertDummyDocuments(MongoCollection<Document> collection) {

        List<Document> list = new ArrayList<Document>();

        Calendar cal = Calendar.getInstance();

        for (int i = 1; i <= 5; i++) {

            Document data = new Document();
            data.append("number", i);
            data.append("name", "mkyong-" + i);
            // data.append("date", cal.getTime());

            // +1 day
            cal.add(Calendar.DATE, 1);

            list.add(data);

        }

        collection.insertMany(list);

    }

    public static void main(String[] args) {

        try {

            MongoClient mongoClient = new MongoClient("localhost", 27017);
            MongoDatabase database = mongoClient.getDatabase("test");

            // get a single collection
            MongoCollection<Document> collection = database.getCollection("dummyColl");

            insertDummyDocuments(collection);

            System.out.println("1. Find first matched document");
            Document document = collection.find().first();
            System.out.println(document);

            System.out.println("\n1. Find all matched documents");
            FindIterable<Document> documents = collection.find();
            MongoCursor<Document> mongoCursor = documents.iterator();
            while (mongoCursor.hasNext()) {
                System.out.println(mongoCursor.next());
            }

            System.out.println("\n1. Get 'name' field only");
//            Document allQuery = new Document();
            Document fields = new Document();
            fields.put("name", 1);

            FindIterable<Document> projection = collection.find().projection(fields);
            MongoCursor<Document> iterator = projection.iterator();
            while (iterator.hasNext()) {
                System.out.println(iterator.next());
            }

            System.out.println("\n2. Find where number = 5");
            Document whereQuery = new Document();
            whereQuery.put("number", 5);
            FindIterable<Document> whereDocuments = collection.find(whereQuery);
            MongoCursor<Document> whereIterator = whereDocuments.iterator();
            while (whereIterator.hasNext()) {
                System.out.println(whereIterator.next());
            }

            System.out.println("\n2. Find where number in 2,4 and 5");
            Document inQuery = new Document();
            List<Integer> list = new ArrayList<Integer>();
            list.add(2);
            list.add(4);
            list.add(5);
            inQuery.put("number", new Document("$in", list));
            FindIterable<Document> listDocuments = collection.find(inQuery);
            MongoCursor<Document> listIterator = listDocuments.iterator();
            while (listIterator.hasNext()) {
                System.out.println(listIterator.next());
            }

            System.out.println("\n2. Find where 5 > number > 2");
            Document gtQuery = new Document();
            gtQuery.put("number", new Document("$gt", 2).append("$lt", 5));
            FindIterable<Document> gtDocuments = collection.find(gtQuery);
            MongoCursor<Document> gtIterator = gtDocuments.iterator();
            while (gtIterator.hasNext()) {
                System.out.println(gtIterator.next());
            }

            System.out.println("\n2. Find where number != 4");
            Document neQuery = new Document();
            neQuery.put("number", new Document("$ne", 4));
            FindIterable<Document> neDocuments = collection.find(neQuery);
            MongoCursor<Document> neIterator = neDocuments.iterator();
            while (neIterator.hasNext()) {
                System.out.println(neIterator.next());
            }

            System.out.println("\n3. Find when number = 2 and name = 'mkyong-2' example");
            Document andQuery = new Document();

            List<Document> obj = new ArrayList<Document>();
            obj.add(new Document("number", 2));
            obj.add(new Document("name", "mkyong-2"));
            andQuery.put("$and", obj);

            System.out.println(andQuery.toString());

            FindIterable<Document> andDocuments = collection.find(andQuery);
            MongoCursor<Document> andIterator = andDocuments.iterator();
            while (andIterator.hasNext()) {
                System.out.println(andIterator.next());
            }

            System.out.println("\n4. Find where name = 'Mky.*-[1-3]', case sensitive example");
            Document regexQuery = new Document();
            regexQuery.put("name",
                    new Document("$regex", "Mky.*-[1-3]")
                            .append("$options", "i"));

            System.out.println(regexQuery.toString());

            FindIterable<Document> regexDocuments = collection.find(regexQuery);
            MongoCursor<Document> regexIterator = regexDocuments.iterator();
            while (regexIterator.hasNext()) {
                System.out.println(regexIterator.next());
            }

            collection.drop();

            System.out.println("Done");

        } catch (MongoException e) {
            e.printStackTrace();
        }

    }
}
```
