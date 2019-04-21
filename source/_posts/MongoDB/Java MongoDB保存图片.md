---
title: Java MongoDB保存图片
date: 2019-04-21
categories: MongoDB
---

本文介绍如何使用**GridFS API**把图片文件保存到MongoDB。GridFS API也能保存其他二进制文件，如视频和音乐文件。

## 1. 保存图片
下面代码使用`photo`命名空间，新的`filename`保存图片到MongoDB。
```
String newFileName = "mkyong-java-image";
File imageFile = new File("mongodb.png");
GridFS gfsPhoto = new GridFS(db, "photo");
GridFSInputFile gfsFile = gfsPhoto.createFile(imageFile);
gfsFile.setFilename(newFileName);
gfsFile.save();
```

## 2. 获取图片
```
String newFileName = "mkyong-java-image";
GridFS gfsPhoto = new GridFS(db, "photo");
GridFSDBFile imageForOutput = gfsPhoto.findOne(newFileName);
System.out.println(imageForOutput);
```

输出，图片以如下的JSON格式被保存：
```
{
	"_id" :
	{
		"$oid" : "4dc9511a14a7d017fee35746"
	} ,
	"chunkSize" : 262144 ,
	"length" : 22672 ,
	"md5" : "1462a6cfa27669af1d8d21c2d7dd1f8b" ,
	"filename" : "mkyong-java-image" ,
	"contentType" :  null  ,
	"uploadDate" :
	{
		"$date" : "2011-05-10T14:52:10Z"
	} ,
	"aliases" :  null
}
```

## 3. 打印所有图片

使用DBCursor遍历所有图片。

```
GridFS gfsPhoto = new GridFS(db, "photo");
DBCursor cursor = gfsPhoto.getFileList();
while (cursor.hasNext()) {
	System.out.println(cursor.next());
}
```

## 4. 保存为另一张图片

从MongoDB中获取图片并把它保存为另一张图片。

```
String newFileName = "mkyong-java-image";
GridFS gfsPhoto = new GridFS(db, "photo");
GridFSDBFile imageForOutput = gfsPhoto.findOne(newFileName);
imageForOutput.writeTo("mongodbNew.png"); //output to new file
```

## 5. 删除图片

```
String newFileName = "mkyong-java-image";
GridFS gfsPhoto = new GridFS(db, "photo");
gfsPhoto.remove(gfsPhoto.findOne(newFileName));
```

## 完整实例

```
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;
import com.mongodb.MongoClient;
import com.mongodb.MongoException;
import com.mongodb.gridfs.GridFS;
import com.mongodb.gridfs.GridFSDBFile;
import com.mongodb.gridfs.GridFSInputFile;

import java.io.File;
import java.io.IOException;
import java.net.UnknownHostException;

public class SaveImage {
    public static void main(String[] args) {

        try {

            MongoClient mongoClient = new MongoClient("localhost", 27017);
            DB db = mongoClient.getDB("imagedb");
            DBCollection collection = db.getCollection("dummyColl");

            String newFileName = "mkyong-java-image";

            File imageFile = new File("mongodb.png");

            // create a "photo" namespace
            GridFS gfsPhoto = new GridFS(db, "photo");

            // get image file from local drive
            GridFSInputFile gfsFile = gfsPhoto.createFile(imageFile);

            // set a new filename for identify purpose
            gfsFile.setFilename(newFileName);

            // save the image file into mongoDB
            gfsFile.save();

            // print the result
            DBCursor cursor = gfsPhoto.getFileList();
            while (cursor.hasNext()) {
                System.out.println(cursor.next());
            }

            // get image file by it's filename
            GridFSDBFile imageForOutput = gfsPhoto.findOne(newFileName);

            // save it into a new image file
            imageForOutput.writeTo("mongodbNew.png");

            // remove the image file from mongoDB
            gfsPhoto.remove(gfsPhoto.findOne(newFileName));

            System.out.println("Done");

        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (MongoException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```
