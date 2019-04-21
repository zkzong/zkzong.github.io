---
title: MongoDB副本集搭建
date: 2019-04-21
categories: MongoDB
---

## 使用命令行

每个副本集成员以下面命令启动：
```
sudo mongod --dbpath /data/db --replSet rs0
```
在其中一台使用***rs.initiate()***命令即可成为主服务。
如果要修改host的名称，可执行如下命令：
```
config={"_id":"rs0","members":[{"_id":0,"host":"192.168.88.129:27017"}]}
rs.reconfig(config,{"force":true})
```
当主服务启动之后，使用mongo登录，然后使用rs.add()命令把另外两台加进来。
```
rs.add("192.168.88.130:27017")
rs.add("192.168.88.132:27017")
```

## 使用config文件

[https://docs.mongodb.com/manual/reference/configuration-options/#replication-options](https://docs.mongodb.com/manual/reference/configuration-options/#replication-options)

使用config文件启动，主要是在`/etc/mongod.conf`文件中配置。
```
replication:
   replSetName: rs0
```
使用`mongod --config /etc/mongod.conf`启动副本集成员。
在其中一台使用`rs.initiate()`命令，则该机器成为主服务。
然后添加其他副本集成员：
```
rs.add("192.168.88.130:27017")
rs.add("192.168.88.132:27017")
```
