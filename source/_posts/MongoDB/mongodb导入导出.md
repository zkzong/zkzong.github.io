---
title: MongoDB导入导出
date: 2019-04-21
categories: MongoDB
---

本文介绍使用`mongoexport`和`mongoimport`命令备份和恢复数据。

## 1. 使用`mongoexport`命令备份数据库

相关命令参数：
```
$ mongoexport --help
Export MongoDB data to CSV, TSV or JSON files.

options:
  -h [ --host ] arg         mongo host to connect to ( <set name>/s1,s2 for
  -u [ --username ] arg     username
  -p [ --password ] arg     password
  -d [ --db ] arg           database to use
  -c [ --collection ] arg   collection to use (some commands)
  -q [ --query ] arg        query filter, as a JSON string
  -o [ --out ] arg          output file; if not specified, stdout is used
```

### 1.1 导出所有文档所有字段到文件`domain-bk.json`文件

```
$ mongoexport -d webmitta -c domain -o domain-bk.json
connected to: 127.0.0.1
exported 10951 records
```

### 1.2 导出所有哦文档的`domain`和`worth`字段

```
$ mongoexport -d webmitta -c domain -f "domain,worth" -o domain-bk.json
connected to: 127.0.0.1
exported 10951 records
```

### 1.3 通过条件导出文档，下面例子导出`worth > 100000`的文档

```
$mongoexport -d webmitta -c domain -f "domain,worth" -q '{worth:{$gt:100000}}' -o domain-bk.json
connected to: 127.0.0.1
exported 10903 records
```

### 1.4 使用用户名和密码导出远程服务器的文档

```
$ mongoexport -h id.mongolab.com:47307 -d heroku_app -c domain -u username123 -p password123 -o domain-bk.json
connected to: id.mongolab.com:47307
exported 10951 records
```

**注意：导出的文档都是json格式**

## 2. 使用`mongoimport`命令恢复数据库

相关命令参数：

```
$ mongoimport --help
connected to: 127.0.0.1
no collection specified!
Import CSV, TSV or JSON data into MongoDB.

options:
  -h [ --host ] arg       mongo host to connect to ( <set name>/s1,s2 for sets)
  -u [ --username ] arg   username
  -p [ --password ] arg   password
  -d [ --db ] arg         database to use
  -c [ --collection ] arg collection to use (some commands)
  -f [ --fields ] arg     comma separated list of field names e.g. -f name,age
  --file arg              file to import from; if not specified stdin is used
  --drop                  drop collection first
  --upsert                insert or update objects that already exist
```

### 2.1 从`domain-bk.json`导入文档，导入的数据库名为`webmitta2`，集合名为`domain2`。不存在的数据库或集合会被自动创建。

```
$ mongoimport -d webmitta2 -c domain2 --file domain-bk.json
connected to: 127.0.0.1
Wed Apr 10 13:26:12 imported 10903 objects
```

### 2.2 导入文档，无则插入，有则更新（根据`_id`）

```
$ mongoimport -d webmitta2 -c domain2 --file domain-bk.json --upsert
connected to: 127.0.0.1
Wed Apr 10 13:26:12 imported 10903 objects
```

### 2.3 使用用户名和密码连接远程服务器，并从本地文件`domain-bk.json`导入文档

```
$ mongoimport -h id.mongolab.com:47307 -d heroku_app -c domain -u username123 -p password123 --file domain-bk.json
connected to: id.mongolab.com:47307
Wed Apr 10 13:26:12 imported 10903 objects
```
