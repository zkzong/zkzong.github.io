---
title: Kafka安装与使用
date: 2019-04-19
categories: Kafka
---

本文假设你是第一次使用Kafka，并且机器上不存在Kafka和ZooKeeper数据。Kafka的命令脚本在Linux和Windows平台上是不一样的，Linux的脚本存放在`bin`目录下，以`.sh`为扩展名，而Windows的脚本存放在`bin/windows`目录下，以`.bat`为扩展名。

## 1. 下载安装包
[下载](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.1.0/kafka_2.11-2.1.0.tgz)2.1.0版本安装包并解压。
```
> tar -xzf kafka_2.11-2.1.0.tgz
> cd kafka_2.11-2.1.0
```

## 2. 启动服务
因为Kafka使用了ZooKeeper，所以在启动Kafka服务之前首先要启动ZooKeeper服务。Kafka安装包自带了ZooKeeper服务，但是只能作为单机使用，在生产环境还是需要搭建ZooKeeper集群。
启动ZooKeeper服务：
```
> bin/zookeeper-server-start.sh config/zookeeper.properties
[2013-04-22 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
```
然后启动Kafka服务：
```
> bin/kafka-server-start.sh config/server.properties
[2013-04-22 15:01:47,028] INFO Verifying properties (kafka.utils.VerifiableProperties)
[2013-04-22 15:01:47,051] INFO Property socket.send.buffer.bytes is overridden to 1048576 (kafka.utils.VerifiableProperties)
...
```

## 3. 创建topic
创建一个叫`test`的topic，只有一个分区和副本。
```
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
运行如下命令可以查看该topic：
```
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```
除了手动创建topic，也可以通过配置broker，当发布的topic不存在时，自动创建topic。

## 4. 发布消息
Kafka自带命令行客户端，可以通过文件输入或标准输入发布消息到Kafka集群。默认每一行是一个单独的消息。

运行producer，在控制台输入消息并发送。
```
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

## 5. 启动consumer
Kafka还有一个命令行consumer把接收到的消息输出。
```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```
如果以上的命令在不同的终端运行，你可以在producer终端输入消息，他们会在不同的consumer终端显示。
所有的命令行工具都有附加选项：运行不带参数的命令会显示该命令的详细使用文档。

## 6. 集群安装
以上是单个broker的安装使用。对Kafka来说，单个broker就是只有一个节点的集群，和启动多个broker实例没什么区别。下面我们把集群的节点数扩展为3个（在同一台机器上）。

首先，为每个broker拷贝配置文件。
```
> cp config/server.properties config/server-1.properties
> cp config/server.properties config/server-2.properties
```
修改相关配置：
```
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dirs=/tmp/kafka-logs-1
 
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dirs=/tmp/kafka-logs-2
```
在集群里每个节点`broker.id`属性是唯一的。因为在一台机器上运行三个节点，所以必须修改端口和日志目录，防止所有的节点注册到一个端口或覆盖彼此的数据。
单机ZooKeeper已经启动，所以只需启动两个新节点：
```
> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
```
创建一个新topic，设置`replication-factor`为3：
```
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```
现在我们有了3个节点的集群，但是怎么知道每个节点在干什么？可以使用`describe topics`命令查看：
```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
```
下面对输出内容作下说明：第一行是所有分区的汇总，后面每一行是一个分区的信息。因为我们该topic只有一个分区，所以只有一行。
- **Leader**是负责分区所有读写的节点。分区内随机选择一个节点作为leader。
- **Replicas**是节点列表，不管它们是不是leader或者是否活跃的它们都会复制分区日志。
- **Isr**是`in-sync`副本集。它是`replicas`的子集，只有活跃的且能被leader捕获的才会在这个子集。

注意：上面例子中，节点1是该topic下分区的leader。

我们可以在原来的topic上运行相同的命令查看它的位置：
```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0
```
可以看到，原来的topic没有副本，在server 0上，这也是集群中唯一的server。

往新topic上发布信息：
```
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```
订阅这些消息：
```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```
现在测试下容错。Broker 1是leader，kill掉它：
```
> ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
> kill -9 7564
```
Windows上使用如下命令：
```
> wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
ProcessId
6016
> taskkill /pid 6016 /f
```
leader切换到slave中的某一个，节点1不再是`in-sync`副本集：
```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```
但是尽管原来写消息的leader挂掉了，消息仍能被订阅。
```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

## 7. 使用Kafka Connect导入/导出数据
从console写入数据和向console写入数据虽然方便，但是可能会有从其他源文件导入或导出数据到其他系统的需求。对于很多系统来说，除了编程的方式，也可以使用Kafka Connect导入/导出数据。
Kafka Connect是Kafka自带的工具，用来导入导出数据。它是运行connectors的扩展工具，通过特殊的逻辑实现和外部系统的交互。下面介绍如何使用Kafka Connect导入/导出数据。

首先，准备测试数据：
```
> echo -e "foo\nbar" > test.txt
```
Windows下可以使用如下命令：
```
> echo foo> test.txt
> echo bar>> test.txt
```
然后，以`standalone`模式启动两个connectors。`standalone`模式就是他们以单独、本地、专一的进程运行。我们提供了三个配置文件作为参数。第一个是Kafka Connect进程的配置，包含通用配置比如Kafka broker
```
> bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```












