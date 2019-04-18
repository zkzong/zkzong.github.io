本文主要介绍mongodb的一些基本操作，如创建、更新、查找、删除记录和创建索引。

## 1. 安装MongoDB

安装可以参考前两篇文章，分别介绍了在Windows和Ubuntu上安装的步骤。

使用`mongod`启动MongoDB

```
$./mongod
Tue Sep 11 21:55:36 [initandlisten] MongoDB starting :
pid=72280 port=27017 dbpath=/data/db/ 64-bit host=Yongs-MacBook-Air.local
Tue Sep 11 21:55:36 [initandlisten] db version v2.0.7, pdfile version 4.5
Tue Sep 11 21:55:36 [initandlisten] options: {}
Tue Sep 11 21:55:36 [initandlisten] journal dir=/data/db/journal
Tue Sep 11 21:55:36 [initandlisten] recover : no journal files present, no recovery needed
Tue Sep 11 21:55:36 [websvr] admin web console waiting for connections on port 28017
Tue Sep 11 21:55:36 [initandlisten] waiting for connections on port 27017
```

## 2. 连接到MongoDB

```
$ ./mongo
MongoDB shell version: 2.0.7
connecting to: test
```

## 3. 创建数据库或集合（表）

MongoDB中，在第一次插入数据时数据库和集合会被自动创建。使用`use database-name`切换到数据库（即使该数据库没有被创建）。

下面例子中，插入数据后，数据库`mkyong`和集合`users`都会被创建。

```
$ ./mongo
MongoDB shell version: 2.0.7
connecting to: test
> use mkyong
switched to db mkyong

> db.users.insert({username:"mkyong",password:"123456"})
> db.users.find()
{ "_id" : ObjectId("504f45cd17f6c778042c3c07"), "username" : "mkyong", "password" : "123456" }
```

你应该知道的三个数据库命令：
1. `show dbs` - 显示所有数据库
2. `use db_name` - 切换到`db_name`数据库
3. `show collections` - 显示当前数据库的所有集合

**在MongoDB中，集合对应SQL中的表。**

## 4. 插入数据

使用`db.tablename.insert({data})`或`db.tablename.save({data})`命令插入数据。

```
> db.users.save({username:"google",password:"google123"})
> db.users.find()
{ "_id" : ObjectId("504f45cd17f6c778042c3c07"), "username" : "mkyong", "password" : "123456" }
{ "_id" : ObjectId("504f48ea17f6c778042c3c0a"), "username" : "google", "password" : "google123" }
```

## 5. 更新数据

使用`db.tablename.update({criteria},{$set: {new value}})`命令更新数据。
下面例子更新了用户名为`mkyong`的密码。

```
> db.users.update({username:"mkyong"},{$set:{password:"hello123"}})
> db.users.find()
{ "_id" : ObjectId("504f48ea17f6c778042c3c0a"), "username" : "google", "password" : "google123" }
{ "_id" : ObjectId("504f45cd17f6c778042c3c07"), "password" : "hello123", "username" : "mkyong" }
```

## 6. 查询数据
使用`db.tablename.find({criteria})`命令查询数据。

### 6.1 查询集合`users`的所有数据
```
> db.users.find()
{ "_id" : ObjectId("504f48ea17f6c778042c3c0a"), "username" : "google", "password" : "google123" }
{ "_id" : ObjectId("504f45cd17f6c778042c3c07"), "password" : "hello123", "username" : "mkyong" }
```

### 6.2 查询username为google的数据
```
> db.users.find({username:"google"})
{ "_id" : ObjectId("504f48ea17f6c778042c3c0a"), "username" : "google", "password" : "google123" }
```

### 6.3 查询username长度小于等于2的数据
```
db.users.find({$where:"this.username.length<=2"})
```

### 6.4 查询包含username字段的数据
```
db.users.find({username:{$exists : true}})
```

## 7. 删除数据

使用`db.tablename.remove({criteria})`命令删除数据。
```
> db.users.remove({username:"google"})
> db.users.find()
{ "_id" : ObjectId("504f45cd17f6c778042c3c07"), "password" : "hello123", "username" : "mkyong" }
```

***注意：***
*删除集合中的所有数据，使用`db.tablename.remove()`*
*删除集合，使用`db.tablename.drop()`*

## 8. 索引

索引能加快查询数据的速度。

### 8.1 查看集合`users`所有索引，默认`_id`是主键并自动创建。
```
> db.users.getIndexes()
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"ns" : "mkyong.users",
		"name" : "_id_"
	}
]
>
```

### 8.2 使用`db.tablename.ensureIndex(column)`命令创建索引

下面例子在`username`列创建索引。
```
> db.users.ensureIndex({username:1})
> db.users.getIndexes()
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"ns" : "mkyong.users",
		"name" : "_id_"
	},
	{
		"v" : 1,
		"key" : {
			"username" : 1
		},
		"ns" : "mkyong.users",
		"name" : "username_1"
	}
]
```

### 8.3 使用`db.tablename.dropIndex(column)`命令删除索引

```
> db.users.dropIndex({username:1})
{ "nIndexesWas" : 2, "ok" : 1 }
> db.users.getIndexes()
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"ns" : "mkyong.users",
		"name" : "_id_"
	}
]
>
```

### 8.4 使用`db.tablename.ensureIndex({column},{unique:true})`命令创建唯一索引

```
> db.users.ensureIndex({username:1},{unique:true});
> db.users.getIndexes()
[
	{
		"v" : 1,
		"key" : {
			"_id" : 1
		},
		"ns" : "mkyong.users",
		"name" : "_id_"
	},
	{
		"v" : 1,
		"key" : {
			"username" : 1
		},
		"unique" : true,
		"ns" : "mkyong.users",
		"name" : "username_1"
	}
]
```

## 9. 帮助

最后，使用`help()`命令提供帮助手册。

### 9.1 `help` - 所有可用命令

```
> help
	db.help()                    help on db methods
	db.mycoll.help()             help on collection methods
	rs.help()                    help on replica set methods
	help admin                   administrative help
	help connect                 connecting to a db help
	help keys                    key shortcuts
	//...
```

### 9.2 `db.help()` - 显示db帮助

```
> db.help()
DB methods:
	db.addUser(username, password[, readOnly=false])
	db.auth(username, password)
	db.cloneDatabase(fromhost)
	db.commandHelp(name) returns the help for the command
	db.copyDatabase(fromdb, todb, fromhost)
	//...
```

### 9.3 `db.collection.help()` - 显示collection帮助

```
> db.users.help()
DBCollection help
	db.users.find().help() - show DBCursor help
	db.users.count()
	db.users.dataSize()
	db.users.distinct( key ) - eg. db.users.distinct( 'x' )
	db.users.drop() drop the collection
	db.users.dropIndex(name)
	//...
```

### 9.4 `db.collection.function.help()` - 显示function帮助

```
> db.users.find().help()
find() modifiers
	.sort( {...} )
	.limit( n )
	.skip( n )
	.count() - total # of objects matching query, ignores skip,limit
	.size() - total # of objects cursor would return, honors skip,limit
	.explain([verbose])
    //...
```
