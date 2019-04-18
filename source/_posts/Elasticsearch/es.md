---
title: ElasticSearch
date: 2019-04-18
categories: ElasticSearch
---

## 入门
Elasticsearch不仅仅是Lucene和全文搜索，我们还能这样去描述它：

+ 分布式的实时文件存储，每个字段都被索引并可被搜索
+ 分布式的实时分析搜索引擎
+ 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

### 安装

安装Marvel
`./bin/plugin -i elasticsearch/marvel/latest`
禁用监控
`echo 'marvel.agent.enabled: false' >> ./config/elasticsearch.yml`
运行
`./bin/elasticsearch`
测试
`curl 'http://localhost:9200/?pretty'`



[https://www.elastic.co/guide/cn/index.html](https://www.elastic.co/guide/cn/index.html)

[http://wiki.jikexueyuan.com/project/elasticsearch-definitive-guide-cn/](http://wiki.jikexueyuan.com/project/elasticsearch-definitive-guide-cn/)

[https://es.xiaoleilu.com/](https://es.xiaoleilu.com/)

[http://www.cnblogs.com/wxw16/tag/Elasticsearch/](http://www.cnblogs.com/wxw16/tag/Elasticsearch/)
