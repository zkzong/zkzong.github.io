本文介绍如何使用`collection.update()`更新文档。

## 测试数据

假设已经插入下面数据：
```
{
	"hosting" : "hostA",
	"type" : "vps",
	"clients" : 1000
},
{
	"hosting" : "hostB",
	"type" : "dedicated server",
	"clients" : 100
},
{
	"hosting" : "hostC",
	"type" : "vps",
	"clients" : 900
}
```
```
{ "_id" : { "$oid" : "id"} , "hosting" : "hostA" , "type" : "vps" , "clients" : 1000}
{ "_id" : { "$oid" : "id"} , "hosting" : "hostB" , "type" : "dedicated server" , "clients" : 100}
{ "_id" : { "$oid" : "id"} , "hosting" : "hostC" , "type" : "vps" , "clients" : 900}
```

## 1. 使用`$set`更新文档

查询`hosting = 'hostB'`的文档并把`clients`的值从`100`更新为`10`。

使用`$set`只更新特定的字段。

```
Document updateDocument = new Document();
updateDocument.append("$set", new Document().append("clients", 110));

Document searchQuery2 = new Document().append("hosting", "hostB");

collection.updateOne(searchQuery2, updateDocument);
```
输出：
```
{ "_id" : { "$oid" : "id"} , "hosting" : "hostA" , "type" : "vps" , "clients" : 1000}
{ "_id" : { "$oid" : "id"} , "hosting" : "hostB" , "type" : "dedicated server" , "clients" : 110}
{ "_id" : { "$oid" : "id"} , "hosting" : "hostC" , "type" : "vps" , "clients" : 900}
```

## 2. 使用`$inc`更新文档

该例子介绍使用`$inc`修改符增加特定值。

查询`hosting = 'hostB'`的文档，并把'clients'的值从100增加到199。
```
Document newDocument2 = new Document().append("$inc",
        new Document().append("clients", 99));

collection.updateOne(new Document().append("hosting", "hostB"), newDocument2);
```
输出：
```
{ "_id" : { "$oid" : "id"} , "hosting" : "hostA" , "type" : "vps" , "clients" : 1000}
{ "_id" : { "$oid" : "id"} , "hosting" : "hostB" , "type" : "dedicated server" , "clients" : 199}
{ "_id" : { "$oid" : "id"} , "hosting" : "hostC" , "type" : "vps" , "clients" : 900}
```

## 3. 使用`updateMany`更新多个文档

该例子介绍使用`updateMany`更新符合条件的多个文档。

查询`type = 'vps'`的文档，并把所有符合条件的文档的clients更新为888。

```
Document updateQuery = new Document();
updateQuery.append("$set", new Document().append("clients", "888"));

Document searchQuery3 = new Document();
searchQuery3.append("type", "vps");

collection.updateMany(searchQuery3, updateQuery);
```
输出：
```
{ "_id" : { "$oid" : "id"} , "hosting" : "hostA" , "clients" : "888" , "type" : "vps"}
{ "_id" : { "$oid" : "id"} , "hosting" : "hostB" , "type" : "dedicated server" , "clients" : 100}
{ "_id" : { "$oid" : "id"} , "hosting" : "hostC" , "clients" : "888" , "type" : "vps"}
```

## 4. 完整实例

```
import com.mongodb.MongoClient;
import com.mongodb.MongoException;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

public class UpdateDocument {
    public static void printAllDocuments(MongoCollection<Document> collection) {
        FindIterable<Document> documents = collection.find();
        MongoCursor<Document> mongoCursor = documents.iterator();
        while (mongoCursor.hasNext()) {
            System.out.println(mongoCursor.next());
        }
    }

    public static void removeAllDocuments(MongoCollection<Document> collection) {
        collection.deleteMany(new Document());
    }

    public static void insertDummyDocuments(MongoCollection<Document> collection) {
        Document document = new Document();
        document.put("hosting", "hostA");
        document.put("type", "vps");
        document.put("clients", 1000);

        Document document2 = new Document();
        document2.put("hosting", "hostB");
        document2.put("type", "dedicated server");
        document2.put("clients", 100);

        Document document3 = new Document();
        document3.put("hosting", "hostC");
        document3.put("type", "vps");
        document3.put("clients", 900);

        collection.insertOne(document);
        collection.insertOne(document2);
        collection.insertOne(document3);
    }

    public static void main(String[] args) {

        try {

            MongoClient mongoClient = new MongoClient("localhost", 27017);
            MongoDatabase database = mongoClient.getDatabase("test");

            // get a single collection
            MongoCollection<Document> collection = database.getCollection("dummyColl");

            System.out.println("\nTesting 1...with $set");

            insertDummyDocuments(collection);

            Document updateDocument = new Document();
            updateDocument.append("$set", new Document().append("clients", 110));

            Document searchQuery2 = new Document().append("hosting", "hostB");

            collection.updateOne(searchQuery2, updateDocument);

            printAllDocuments(collection);
            removeAllDocuments(collection);

            System.out.println("\nTesting 2... with $inc");
            insertDummyDocuments(collection);
            // find hosting = hostB and increase it's "clients" value by 99
            Document newDocument2 = new Document().append("$inc",
                    new Document().append("clients", 99));

            collection.updateOne(new Document().append("hosting", "hostB"), newDocument2);

            printAllDocuments(collection);
            removeAllDocuments(collection);

            System.out.println("\nTesting 3... with $multi");

            insertDummyDocuments(collection);
            // find type = vps , update all matched documents , clients value to 888
            Document updateQuery = new Document();
            updateQuery.append("$set", new Document().append("clients", "888"));

            Document searchQuery3 = new Document();
            searchQuery3.append("type", "vps");

            collection.updateMany(searchQuery3, updateQuery);
            // collection.update(searchQuery3, updateQuery, false, true);

            printAllDocuments(collection);
            removeAllDocuments(collection);

            System.out.println("Done");

        } catch (MongoException e) {
            e.printStackTrace();
        }

    }
}
```
