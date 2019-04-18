本文介绍如何使用`collection.delete()`删除文档。

## 测试数据

插入10个文档：
```
for (int i = 1; i <= 10; i++) {
    collection.insertOne(new Document().append("number", i));
}
```

## 1. collection.delete()

下面是几个删除文档的例子。

### 例1
获取第一个文档并删除。本例中`number = 1`的文档被删除。
```
Document doc = collection.find().first(); //get first document
collection.deleteOne(doc);
```

### 例2
把查询放到Document中。本例中`number = 2`的文档被删除。
```
Document document = new Document();
document.put("number", 2);
collection.deleteOne(document);
```

**两种常见错误：**
**1. 这样的查询只删除`number = 3`的文档**
```
Document document = new Document();
document.put("number", 2);
        document.put("number", 3); //override above value 2
collection.deleteOne(document);
```
**2. 但是如下面的查询，删除并不起作用**
```
Document document = new Document();
List<Integer> list = new ArrayList<Integer>();
list.add(7);
list.add(8);
document.put("number", list);
collection.remove(document);
```
**对于and查询，需要使用`$in`或`$and`操作符，参考例5。**

### 例3
直接使用`Document`删除。本例中`number = 3`的文档被删除。
```
collection.deleteOne(new Document().append("number", 3));
```

### 例4
在`Document`中使用`$gt`操作符。本例中`number = 10`的文档被删除。
```
Document query = new Document();
query.put("number", new Document("$gt", 9));
collection.deleteOne(query);
```

### 例5
使用`$in`构建查询并删除多个文档。本例中`number = 4`和`number = 5`的文档被删除。
```
Document query2 = new Document();
List<Integer> list = new ArrayList<Integer>();
list.add(4);
list.add(5);
query2.put("number", new Document("$in", list));
collection.deleteMany(query2);
```

### 例6
使用循环的方式删除所有文档。（不推荐这种方式，推荐例7的方式）
```
FindIterable<Document> deleteDocuments = collection.find();
MongoCursor<Document> deleteIterator = deleteDocuments.iterator();
while (deleteIterator.hasNext()) {
    collection.deleteOne(deleteIterator.next());
}
```

### 例7
传入一个空的Document对象，删除所有文档。
```
collection.deleteMany(new Document());
```

### 例8
删除文档和集合。
```
collection.drop();
```

### 例9
delete()方法会返回`DeleteResult`对象，它包含了一些关于删除操作的有用信息。可以使用`getDeletedCount()`获取删除文档数。
```
DeleteResult deleteResult = collection.deleteMany(new Document());
System.out.println("删除文档数：" + deleteResult.getDeletedCount());
```

## 2. 完整实例

```
import com.mongodb.MongoClient;
import com.mongodb.MongoException;
import com.mongodb.client.FindIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.MongoDatabase;
import com.mongodb.client.result.DeleteResult;
import org.bson.Document;

import java.util.ArrayList;
import java.util.List;

public class RemoveDocument {
    public static void main(String[] args) {

        try {

            MongoClient mongoClient = new MongoClient("localhost", 27017);
            MongoDatabase database = mongoClient.getDatabase("test");

            // get a single collection
            MongoCollection<Document> collection = database.getCollection("dummyColl");

            //insert number 1 to 10 for testing
            for (int i = 1; i <= 10; i++) {
                collection.insertOne(new Document().append("number", i));
            }

            //remove number = 1
            Document doc = collection.find().first(); //get first document
            collection.deleteOne(doc);

            //remove number = 2
            Document document = new Document();
            document.put("number", 2);
            collection.deleteOne(document);

            //remove number = 3
            collection.deleteOne(new Document().append("number", 3));

            //remove number > 9 , means delete number = 10
            Document query = new Document();
            query.put("number", new Document("$gt", 9));
            collection.deleteOne(query);

            //remove number = 4 and 5
            Document query2 = new Document();
            List<Integer> list = new ArrayList<Integer>();
            list.add(4);
            list.add(5);
            query2.put("number", new Document("$in", list));
            collection.deleteOne(query2);

            FindIterable<Document> deleteDocuments = collection.find();
            MongoCursor<Document> deleteIterator = deleteDocuments.iterator();
            while (deleteIterator.hasNext()) {
                collection.deleteOne(deleteIterator.next());
            }

            collection.deleteMany(new Document());

            collection.drop();

            DeleteResult deleteResult = collection.deleteMany(new Document());
            System.out.println("删除文档数：" + deleteResult.getDeletedCount());

            //print out the document
            FindIterable<Document> documents = collection.find();
            MongoCursor<Document> iterator = documents.iterator();
            while (iterator.hasNext()) {
                System.out.println(iterator.next());
            }

            collection.drop();

            System.out.println("Done");

        } catch (MongoException e) {
            e.printStackTrace();
        }

    }
}
```
