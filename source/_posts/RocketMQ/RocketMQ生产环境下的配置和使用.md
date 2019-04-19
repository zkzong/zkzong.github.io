---
title: RocketMQ生产环境下的配置和使用
date: 2019-04-19
categories: RocketMQ
---

本文的目的是带领读者快速将RocketMQ应用到生产环境中，因此不会探究原理和细节。本文会先介绍RocketMQ的各个角色，然后介绍如何搭建一个高可用的分布式消息队列集群，以及RocketMQ的Consumer和Producer的使用方法与常用命令。

## 1. RocketMQ各部分角色介绍﻿

RocketMQ由四部分组成，先来直观地了解一下这些角色以及各自的功能。分布式消息队列是用来高效地传输消息的，它的功能和现实生活中的邮局收发信件很类似，我们类比地说一下相应的模块。现实生活中的邮政系统要正常运行，离不开下面这四个角色，一是发信者，二是收信者，三是负责暂存、传输的邮局，四是负责协调各个地方邮局的管理机构。对应到RocketMQ中，这四个角色就是Producer、Consumer、Broker和NameServer。

启动RocketMQ的顺序是先启动NameServer，再启动Broker，这时候消息队列已经可以提供服务了，想发送消息就使用Producer来发送，想接收消息就使用Consumer来接收。很多应用程序既要发送，又要接收，可以启动多个Producer和Consumer来发送多种消息，同时接收多种消息。﻿

为了消除单点故障，增加可靠性或增大吞吐量，可以在多台机器上部署多个NameServer和Broker，为每个Broker部署一个或多个Slave。

![RocketMQ各个角色间关系﻿](https://upload-images.jianshu.io/upload_images/292448-3872b423b2cdd4b2.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

了解了四种角色以后，再介绍一下Topic和Message Queue这两个名词。一个分布式消息队列中间件部署好以后，可以给很多个业务提供服务，同一个业务也有不同类型的消息要投递，这些不同类型的消息以不同的Topic名称来区分。所以发送和接收消息前，先创建Topic，针对某个Topic发送和接收消息。有了Topic以后，还需要解决性能问题。如果一个Topic要发送和接收的数据量非常大，需要能支持增加并行处理的机器来提高处理速度，这时候一个Topic可以根据需求设置一个或多个Message Queue，Message Queue类似分区或Partition。Topic有了多个Message Queue后，消息可以并行地向各个Message Queue发送，消费者也可以并行地从多个Message Queue读取消息并消费。

## 2. 多机集群配置和部署﻿

本节将说明如何只用两台物理机，搭建出双主、双从、无单点故障的高可用RocketMQ集群。假设这两台物理机的IP分别是192.168.100.131和192.168.100.132。

### 2.1 启动多个NameServer和Broker﻿

首先在这两台机器上分别启动NameServer（nohup sh bin/mqnamesrv &），这样我们就得到了一个无单点的NameServer服务，服务地址是`192.168.100.131:9876;192.168.100.132:9876`。﻿

然后启动Broker，每台机器上都要分别启动一个Master角色的Broker和一个Slave角色的Broker，并互为主备。可以基于RocketMQ自带的示例配置文件写自己的配置文件（示例配置文件在conf/2m-2s-sync目录下）。﻿

1）192.168.100.131机器上Master Broker的配置文件：﻿
```
namesrvAddr=192.168.100.131:9876;192.168.100.132:9876
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
listenPort=10911
storePathRootDir=/home/rocketmq/store-a
```
2）192.168.100.132机器上Master Broker的配置文件：﻿
```
namesrvAddr=192.168.100.131:9876;192.168.100.132:9876
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH
listenPort=10911
storePathRootDir=/home/rocketmq/store-b
```
3）192.168.100.131机器上SlaveBroker的配置文件：﻿
```
namesrvAddr=192.168.100.131:9876;192.168.100.132:9876
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
listenPort=11011
storePathRootDir=/home/rocketmq/store-a
```
4）192.168.100.132机器上Slave Broker的配置文件：﻿
```
namesrvAddr=192.168.100.131:9876;192.168.100.132:9876
brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=1
deleteWhen=04
fileReservedTime=48
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
listenPort=11011
storePathRootDir=/home/rocketmq/store-b
```
然后分别使用如下命令启动四个Broker：﻿
```
nohup sh ./bin/mqbroker -c config_file &
```

这样一个高可用的RocketMQ集群就搭建好了，还可以在一台机器上启动rocketmq-console，比如在192.168.100.131上启动RocketMQ-console，然后在浏览器中输入地址`192.168.100.131:8080`，这样就可以可视化地查看集群状态了。

### 2.2 配置参数介绍﻿

本节将逐个介绍Broker配置文件中用到的参数含义：﻿

**1）namesrvAddr=192.168.100.131:9876;192.168.100.132:9876**

NamerServer的地址，可以是多个。﻿

**2）brokerClusterName=DefaultCluster**

Cluster的地址，如果集群机器数比较多，可以分成多个Cluster，每个Cluster供一个业务群使用。﻿

**3）brokerName=broker-a**

Broker的名称，Master和Slave通过使用相同的Broker名称来表明相互关系，以说明某个Slave是哪个Master的Slave。﻿

**4）brokerId=0**

一个Master Borker可以有多个Slave，0表示Master，大于0表示不同Slave的ID。﻿

**5）fileReservedTime=48**

在磁盘上保存消息的时长，单位是小时，自动删除超时的消息。﻿

**6）deleteWhen=04**

与fileReservedTime参数呼应，表明在几点做消息删除动作，默认值04表示凌晨4点。﻿

**7）brokerRole=SYNC_MASTER**

brokerRole有3种：SYNC_MASTER、ASYNC_MASTER、SLAVE。关键词SYNC和ASYNC表示Master和Slave之间同步消息的机制，SYNC的意思是当Slave和Master消息同步完成后，再返回发送成功的状态。﻿

**8）flushDiskType=ASYNC_FLUSH**

flushDiskType表示刷盘策略，分为SYNC_FLUSH和ASYNC_FLUSH两种，分别代表同步刷盘和异步刷盘。同步刷盘情况下，消息真正写入磁盘后再返回成功状态；异步刷盘情况下，消息写入page_cache后就返回成功状态。﻿

**9）listenPort=10911**

Broker监听的端口号，如果一台机器上启动了多个Broker，则要设置不同的端口号，避免冲突。﻿

**10）storePathRootDir=/home/rocketmq/store-a**

存储消息以及一些配置信息的根目录。﻿

这些配置参数，在Broker启动的时候生效，如果启动后有更改，要重启Broker。现在使用云服务或多网卡的机器比较普遍，Broker自动探测获得的ip地址可能不符合要求，通过brokerIP1=47.98.41.234这样的配置参数，可以设置Broker机器对外暴露的ip地址。

### 2.3 发送/接收消息示例﻿

可以用自己熟悉的开发工具创建一个Java项目，加入RocketMQ Client包的依赖，用下面代码发送消息，这个示例代码是以Sync方式发送消息的。﻿

**Producer示例程序﻿**

```java
public class SyncProducer {
    public static void main(String[] args) throws Exception {
        //Instantiate with a producer group name.
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        // Specify name server addresses.
        producer.setNamesrvAddr("localhost:9876");
        //Launch the instance.
        producer.start();
        for (int i = 0; i < 100; i++) {
            //Create a message instance, specifying topic, tag and message body.
            Message msg = new Message("TopicTest" /* Topic */,
                    "TagA" /* Tag */,
                    ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            //Call send message to deliver message to one of brokers.
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        //Shut down once the producer instance is not longer in use.
        producer.shutdown();
    }
}
```

主要流程是：创建一个DefaultMQProducer对象，设置好GroupName和NameServer地址后启动，然后把待发送的消息拼装成Message对象，使用Producer来发送。
接下来看看如何接收消息，也就是使用DefaultMQPushConsumer类实现的消费者程序，代码如下。﻿

**Consumer示例程序﻿**

```java
public class Consumer {

    public static void main(String[] args) throws InterruptedException, MQClientException {

        // Instantiate with specified consumer group name.
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name");

        // Specify name server addresses.
        consumer.setNamesrvAddr("localhost:9876");

        // Subscribe one more more topics to consume.
        consumer.subscribe("TopicTest", "*");
        // Register callback to execute on arrival of messages fetched from brokers.
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        //Launch the consumer instance.
        consumer.start();

        System.out.printf("Consumer Started.%n");
    }
}
```

Consumer或Producer都必须设置GroupName、NameServer地址以及端口号。然后指明要操作的Topic名称，最后进入发送和接收逻辑。

### 2.4 常用管理命令﻿

MQAdmin是RocketMQ自带的命令行管理工具，在bin目录下，运行mqadmin即可执行。使用mqadmin命令，可以进行创建、修改Topic，更新Broker的配置信息，查询特定消息等各种操作。

具体命令参考：https://rocketmq.apache.org/docs/cli-admin-tool/

### 2.5 通过图形界面管理集群﻿

对于RocketMQ新手，可以启动运维服务，从页面上直观看到消息队列集群的状态。有一定经验以后，可以使用命令行更快捷，其功能更全面。﻿

运维服务程序是个SpringBoot项目，需要从GitHub上的apache/rocketmq-externals里下载源码（https://github.com/apache/rocketmq-externals/tree/master/rocketmq-console）。﻿

进入下载源码的目录，运行如下命令即可启动：﻿
```
mvn spring-boot:run
```
也可以编译成jar包，通过java -jar来执行。﻿

服务启动后，在浏览器里访问server_ip_address:8080（server_ip_address是启动rocketmq-console的机器IP）地址就可看到集群的状态。

![rocketmq-console页面](https://upload-images.jianshu.io/upload_images/292448-4df74f9cf252ea06.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

### 2.6 本章小结﻿

在生产环境中使用RocketMQ集群需要比QuickStart部分了解更多的内容，本文在机器角色、集群配置和部署，以及集群管理方面都做了介绍，用户可以基于这些内容搭建起一个生成环境的RocketMQ消息队列集群，在数据量不大的非关键场景，可以快速上线。
