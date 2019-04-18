---
title: Zookeeper安装
date: 2019-04-18
categories: Dubbo
---

## 单机安装
### 1. 安装jdk和zk
### 2. 创建配置文件
```
cp zoo_sample.cfg zoo.cfg
```
### 3. 把zk加入环境变量
```
vim /etc/profile
export PATH=$PATH:/opt/zookeeper/bin
```
### 4. 执行source命令是修改的环境变量生效
```
source /etc/profile
env
```

## 集群搭建
### 1. 修改配置文件zoo.cfg
```
server.1=192.168.88.129:2888:3888
server.2=192.168.88.130:2888:3888
server.3=192.168.88.132:2888:3888
```
### 2. 创建相关目录
数据目录和日志目录，在zoo.cfg中配置
默认的只有数据目录dataDir
### 3. 创建ServerID标识
在dataDir目录下配置myid文件
这个文件里面有一个数据就是server.1=192.168.88.129:2888:3888中的1。每个服务器对应相应的数字。
```
echo "1" > /opt/zookeeper/myid
echo "2" > /opt/zookeeper/myid
echo "3" > /opt/zookeeper/myid
```
### 4. 重启服务并查看状态
```
zkServer.sh restart
zkServer.sh status
```
### 5. 连接zk集群
```
zkCli -server 192.168.88.129:2181
```
