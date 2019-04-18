## 简介

对MongoDB进行访问控制就是在访问之前先对用户校验，只有当用户有相关权限是才能根据角色执行相关操作。

MongoDB支持各种认证机制，具体请查看[Authentication Mechanisms](https://docs.mongodb.com/manual/core/authentication-mechanisms/)。

下面使用单独的mongod实例和默认的认证机制说明访问控制。

## 复制集和集群

当访问控制可用时，复制集和集群需要内部认证。详细介绍请查看[Internal Authentication](https://docs.mongodb.com/manual/core/security-internal-authentication/)。

## 管理员

当访问控制可用时，需要保证在admin数据库中有一个用户有`userAdmin`或`userAdminAnyDatabase`角色。这个用户可以管理用户和角色，如创建用户，授权或撤销角色，创建或修改自定义角色。

## 步骤

### 1. 以没有访问控制的方式启动MongoDB

如，下面命令以没有访问控制的方式启动一个单独的mongod实例：

`mongod --port 27017 --dbpath /data/db1`

### 2. 连接到实例

`mongo --port 27017`

可以指定相应的参数连接到mongo，如--host。

### 3. 创建管理员用户

在admin数据库中添加一个`userAdminAnyDatabase`角色的用户。例如，下面命令在admin数据库中创建了myUserAdmin用户。

**注意：你创建用户的数据库（如admin）是用户认证数据库。虽然用户认证了该数据库，但还可以拥有其他数据库的角色。**
```
use admin
db.createUser(
  {
    user: "myUserAdmin",
    pwd: "abc123",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```

### 4. 使用访问控制重启MongoDB

使用参数`--auth`重启mongod。如果使用了配置文件，那就是`security.authorization`。

`mongod --auth --port 27017 --dbpath /data/db1`

### 5. 使用管理员账号连接并认证

通过mongo shell，有两种连接方式：

+ 1、 通过用户信息认证连接
+ 2、 不认证连接，然后使用`db.auth()`方法认证

#### 连接时认证

开启mongo shell，使用`-u <username>`，`-p <password>`和`--authenticationDatabase <database>`参数

`mongo --port 27017 -u "myUserAdmin" -p "abc123" --authenticationDatabase "admin"`

#### 连接后认证

`mongo --port 27017`

切换到认证数据库（如admin），然后使用`db.auth(<username>, <pwd>)`方法认证：

```
use admin
db.auth("myUserAdmin", "abc123" )
```

### 6. 创建其他用户

一旦被认证为管理员，就可以使用`db.createUser()`创建其他用户，可以分配内置角色或自定义角色给用户。

用户`myUserAdmin`只有管理用户和角色的权限。当用户`myUserAdmin`试图执行其他操作，如从test数据库的foo集合中读取数据，MongoDB就会报错。

```
use test
db.createUser(
  {
    user: "myTester",
    pwd: "xyz123",
    roles: [ { role: "readWrite", db: "test" },
             { role: "read", db: "reporting" } ]
  }
)
```

### 7. 使用myTester连接并认证

#### 连接时认证

`mongo --port 27017 -u "myTester" -p "xyz123" --authenticationDatabase "test"`

#### 连接后认证

```
mongo --port 27017
use test
db.auth("myTester", "xyz123" )
```

#### 插入数据

用户myTester对test数据库有读写权限，对reporting数据库有读权限。如，可以对test数据库执行插入操作：

`db.foo.insert( { x: 1, y: 1 } )`
