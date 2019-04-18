---
title: ElasticSearch入门
date: 2019-04-18
categories: ElasticSearch
---

ElasticSearch是个开源的基于企业级REST的实时搜索和分析引擎。它的核心搜索功能是建立在Apache Lucene之上，而且支持许多其他特性。

它是由Java语言编写的，支持存储，索引，搜索和分析实时数据。和MongoDB类似，ElasticSearch也是个基于文档的NoSQL数据存储。

ElasticSearch特性：
+ 开源
+ 支持全文本简单、强大的搜索
+ 支持基于REST的API（基于HTTP的JSON）
+ 支持实时搜索分析
+ 根据定义，它是分布式的
+ 支持多租赁特性
+ 支持云和大数据环境
+ 支持跨平台
+ 非规范化的NoSQL数据存储

ElasticSearch的优势：
+ 开源
+ 轻量级的REST API
+ 高可用性，高扩展性
+ 支持缓存数据
+ 无模式
+ 快速搜索性能
+ 支持结构化和非结构化的数据
+ 支持分布式，分片，复制，集群和多节点架构
+ 支持批量操作
+ 建立实时的图表和仪表盘

ElasticSearch的劣势：
+ 不支持MapReduce操作
+ 不能作为主数据存储
+ 不兼容ACID的数据存储
+ 不支持事务和分布式事务
+ 没有内置的认证和授权机制

使用ElasticSearch的公司：
+ Github.com, Quora.com, Stackoverflow.com
+ eBay, DELL, Cisco, Mozilla, Wikimedia
+ Netflix, Symatics, Facebook
+ UK HMRC

例如，Github.com使用ElasticSearch搜索文件，历史等。大部分公司使用ELK管理日志，监控系统。ELK即ElasticSearch、Logstash和Kibana。

## 1. 安装ElasticSearch
**ElasticSearch使用Java编写，所以需要先安装JDK并设置环境变量。**
要在本地安装ElasticSearch，按照下列步骤。

### 1.1 从[https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)下载ElasticSearch。

### 1.2 Windows
+ 下载并解压Zip文件：elasticsearch-5.2.1.zip
+ 加压后的文件放到F:\elasticsearch-5.2.1
![image.png](https://upload-images.jianshu.io/upload_images/292448-050e92186ef3cc64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
+ 设置环境变量
```
PATH = F:\elasticsearch-5.2.1\bin
```
+ 启动ElasticSearch
```
F:/>elasticsearch.bat
```
![image.png](https://upload-images.jianshu.io/upload_images/292448-668e58e8507b647b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
+ 在浏览器中输入`http://localhost:9200`访问ElasticSearch。可以在命令行中使用`Ctrl + C`停止ElasticSearch服务。

### 1.3 Ubuntu：使用tar文件安装
+ 下载并解压tar文件
```
tar -xvf elasticsearch-5.2.1.tar.gz
```
+ 启动ElasticSearch
```
$ ./elasticsearch
```
+ 在浏览器中输入`http://localhost:9200`访问ElasticSearch

### 1.4 Ubuntu：使用命令行安装
+ 执行以下命令下载ElasticSearch
```
$ sudo wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.1.deb
```
下载ElasticSearch DEB文件：elasticsearch-5.2.1.deb
+ 执行下面dpkg命令安装ElasticSearch
```
$ sudo dpkg -i elasticsearch-5.2.1.deb
```
ElasticSearch默认安装在`/usr/share/elasticsearch`。
+ 启动ElasticSearch
```
$ ./elasticsearch
```
+ 在浏览器中输入`http://localhost:9200`访问ElasticSearch

**ElasticSearch默认端口是9200，可以根据需要修改**

### 1.5 ElasticSearch启动后，访问默认URL，可以得到如下响应报文：
```
{
  "name" : "rBvi0Hs",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "kOQQ_nqfTW-b4vQ00XSvdg",
  "version" : {
    "number" : "5.2.1",
    "build_hash" : "db0d481",
    "build_date" : "2017-02-09T22:05:32.386Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
  "tagline" : "You Know, for Search"
}
```

**源码地址：[https://github.com/elastic/elasticsearch](https://github.com/elastic/elasticsearch)**

## 2. ElasticSearch REST API URL基础
ElasticSearch REST API URL应该遵循如下格式。
![image.png](https://upload-images.jianshu.io/upload_images/292448-ad0ce1919b5f5458.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
+ Server可以是任意的服务器名称或主机名，例如“myserver”。有时使用Node + Port的方式，如“”
## 3. ElasticSearch术语

## 4. ElasticSearch命令基础

## 5. ElasticSearch CRUD操作
## 6. 索引必须小写

