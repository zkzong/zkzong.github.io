本文介绍如何安装配置单机版的RocketMQ，以及简单地收发消息。读者也可以参考RocketMQ官网的说明文档。

## 1. RocketMQ的下载、安装和配置
RocketMQ的Binary版是一些编译好的jar和辅助的shell脚本，可以直接从官网找到下载链接（http://rocketmq.apache.org/dowloading/releases/），也可以下载源码自己编译。
系统要求：64bit的Linux、Unix或Mac。Java版本大于等于JDK1.8。如果需要从GitHub上下载源码和编译的话，需要安装Maven 3.2.x和Git。
RocketMQ当前的最新版本是4.2.0，下面以Binary版本为例说明如何快速使用：
```
> unzip rocketmq-all-4.2.0-bin-release.zip -d ./rocketmq-all-4.2.0-bin
> ls
> cd rocketmq-all-4.2.0-bin/
```
里面含有以下内容：
```
LICENSE  NOTICE  README.md  benchmark/  bin/   conf/  lib/
```
LICENSE、NOTICE和README.md包括一些版权声明和功能说明信息；benchmark里包括运行benchmark程序的shell脚本；bin文件夹里含有各种使用RocketMQ的shell脚本（Linux平台）和cmd脚本（Windows平台），比如常用的启动NameServer的脚本mqnamesrv，启动Broker的脚本mqbroker，集群管理脚本mqadmin等；conf文件夹里有一些示例配置文件，包括三种方式的broker配置文件、logback日志配置文件等，用户在写配置文件的时候，一般基于这些示例配置文件，加上自己特殊的需求即可；lib文件夹里包括RocketMQ各个模块编译成的jar包，以及RocketMQ依赖的一些jar包，比如Netty、commons-lang、FastJSON等。

## 2. 启动消息队列服务
启动单机的消息队列服务比较简单，不需要写配置文件，只需要依次启动本机的NameServer和Broker即可。
启动NameServer：
```
> nohup sh bin/mqnamesrv &
> tail -f ~/logs/rocketmqlogs/namesrv.log
The Name Server boot success...
```
启动Broker：
```
> nohup sh bin/mqbroker -n localhost:9876 &
> tail -f ~/logs/rocketmqlogs/broker.log
The broker[%s, 192.168.0.233:10911] boot success...
```
这种方式启动Broker，Topic并不会自动创建。如果需要自动创建，使用命令`nohup sh bin/mqbroker -n localhost:9876 autoCreateTopicEnable=true &`，添加了`autoCreateTopicEnable=true`。当然也可以使用mqadmin或者rocketmq-console添加Topic。

## 3. 用命令行发送和接收消息
为了快速展示发送和接收消息，本节展示的是用命令行发送和接收消息，实际上就是运行写好的demo程序，后续我们可以参考这些demo来写自己的发送和接收程序。
运行示例程序，发送和接收消息：
```
> export NAMESRV_ADDR=localhost:9876
> sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
SendResult [sendStatus=SEND_OK, msgId= ...
> sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

## 4. 关闭消息队列
消息队列被启动后，如果不主动关闭，则会一直在后台运行，占用系统资源。我们有专门用来关闭NameServer和Broker的命令。
关闭NameServer和Broker：
```
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```
恭喜，现在你已经能够使用RocketMQ发送并接收消息了，使用消息队列的基本功能就是这么简单。
