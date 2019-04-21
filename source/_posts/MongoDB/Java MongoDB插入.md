---
title: Java MongoDB插入
date: 2019-04-21
categories: MongoDB
---

本文介绍3中插入文档的方法。

## 1. 使用Document插入文档

```
Document document = new Document();
document.put("database", "test");
document.put("table", "hosting");

Document documentDetail = new Document();
documentDetail.put("records", 99);
documentDetail.put("index", "vps_index1");
documentDetail.put("active", "true");
document.put("detail", documentDetail);

collection.insertOne(document);
```

## 2. 使用Map插入文档

```
Map<String, Object> documentMap = new HashMap<String, Object>();
documentMap.put("database", "test");
documentMap.put("table", "hosting");

Map<String, Object> documentMapDetail = new HashMap<String, Object>();
documentMapDetail.put("records", 99);
documentMapDetail.put("index", "vps_index1");
documentMapDetail.put("active", "true");

documentMap.put("detail", documentMapDetail);

collection.insertOne(new Document(documentMap));
```

## 3. 使用JSON插入文档

```
String json = "{\"database\":\"test\", \"table\":\"hosting\", \"detail\":{\"records\":99, \"index\":\"vps_index1\", \"active\":\"true\"}}";
Document d = Document.parse(json);

collection.insertOne(d);
```

## 完整实例
```
import com.mongodb.MongoClient;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import java.util.HashMap;
import java.util.Map;

public class InsertDocument {
    public static void main(String[] args) {
        MongoClient mongoClient = new MongoClient("localhost", 27017);
        MongoDatabase database = mongoClient.getDatabase("test");

        MongoCollection<Document> collection = database.getCollection("coll");

        Document document = new Document();
        document.put("database", "test");
        document.put("table", "hosting");

        Document documentDetail = new Document();
        documentDetail.put("records", 99);
        documentDetail.put("index", "vps_index1");
        documentDetail.put("active", "true");
        document.put("detail", documentDetail);

        collection.insertOne(document);

        Map<String, Object> documentMap = new HashMap<String, Object>();
        documentMap.put("database", "test");
        documentMap.put("table", "hosting");

        Map<String, Object> documentMapDetail = new HashMap<String, Object>();
        documentMapDetail.put("records", 99);
        documentMapDetail.put("index", "vps_index1");
        documentMapDetail.put("active", "true");

        documentMap.put("detail", documentMapDetail);

        collection.insertOne(new Document(documentMap));

        String json = "{\"database\":\"test\", \"table\":\"hosting\", \"detail\":{\"records\":99, \"index\":\"vps_index1\", \"active\":\"true\"}}";
        Document d = Document.parse(json);

        collection.insertOne(d);

        FindIterable<Document> documents = collection.find();
        MongoCursor<Document> mongoCursor = documents.iterator();
        while (mongoCursor.hasNext()) {
            System.out.println(mongoCursor.next());
        }
    }
}
```
