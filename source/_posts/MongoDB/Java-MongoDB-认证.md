默认情况下，MongoDB不需要用户名和密码即可运行。本文介绍如何使用MongoDB驱动在安全模式下连接数据库。

## 1. 安全模式下启动MongoDB

使用`--auth`参数启动MongoDB，然后需要用户名和密码才能对数据库进行操作。

```
mongod --auth
```

添加相应的数据库和用户：

```
> use admin
> db.addUser("admin","password")
> use testdb
> db.addUser("mkyong","password")
```

具体可以参考之前文章[MongoDB认证](http://blog.csdn.net/zongzhankui/article/details/78380582)

## 2. Java MongoDB认证

```
import com.mongodb.MongoClient;
import com.mongodb.MongoCredential;
import com.mongodb.ServerAddress;
import com.mongodb.client.MongoDatabase;

import java.util.ArrayList;
import java.util.List;

public class Authentication {
    public static void main(String[] args) {
        try {
            //连接到MongoDB服务 如果是远程连接可以替换“localhost”为服务器所在IP地址
            //ServerAddress()两个参数分别为 服务器地址 和 端口
            ServerAddress serverAddress = new ServerAddress("localhost",27017);
            List<ServerAddress> addrs = new ArrayList<ServerAddress>();
            addrs.add(serverAddress);

            //MongoCredential.createScramSha1Credential()三个参数分别为 用户名 数据库名称 密码
            MongoCredential credential = MongoCredential.createScramSha1Credential("username", "databaseName", "password".toCharArray());
            List<MongoCredential> credentials = new ArrayList<MongoCredential>();
            credentials.add(credential);

            //通过连接认证获取MongoDB连接
            MongoClient mongoClient = new MongoClient(addrs,credentials);

            //连接到数据库
            MongoDatabase mongoDatabase = mongoClient.getDatabase("databaseName");
            System.out.println("Connect to database successfully");
        } catch (Exception e) {
            System.err.println( e.getClass().getName() + ": " + e.getMessage() );
        }
    }
}
```
