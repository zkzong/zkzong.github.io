---
title: Redis哨兵模式安装与配置
date: 2019-04-19
categories: Redis
---

在介绍哨兵模式之前首先介绍下**Redis主从复制**。

`Redis` **主从复制** 可将 **主节点** 数据同步给 **从节点**，从节点此时有两个作用：
1. 一旦 **主节点宕机**，**从节点** 作为 **主节点** 的 **备份** 可以随时顶上来。
2. 扩展 **主节点** 的 **读能力**，分担主节点读压力。

![Redis主从复制模式](https://upload-images.jianshu.io/upload_images/292448-939b17a1363404f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**主从复制** 同时存在以下几个问题：
1. 一旦 **主节点宕机**，**从节点** 晋升成 **主节点**，同时需要修改 **应用方** 的 **主节点地址**，还需要命令所有 **从节点** 去 **复制** 新的主节点，整个过程需要 **人工干预**。
2. **主节点** 的 **写能力** 受到 **单机的限制**。
3. **主节点** 的 **存储能力** 受到 **单机的限制**。
4. **原生复制** 的弊端在早期的版本中也会比较突出，比如：`Redis` **复制中断** 后，**从节点** 会发起 `psync`。此时如果 **同步不成功**，则会进行 **全量同步**，**主库** 执行 **全量备份** 的同时，可能会造成毫秒或秒级的 **卡顿**。

---

Redis Sentinel 是 Redis 高可用 的实现方案。Sentinel 是一个管理多个 Redis 实例的工具，它可以实现对 Redis 的 监控、通知、自动故障转移。

## 1. Redis Sentinel规划
一个一主多从的Redis系统中，可以使用多个哨兵进行监控任务以保证系统足够稳健。此时，不仅哨兵会同时监控主数据库和从数据库，哨兵之间也会相互监控。在这里，建议大家哨兵至少部署3个，并且使用奇数个哨兵。
本文使用3台服务器作为sentinal哨兵节点，另外3台服务器作为主从数据节点。

|IP|端口号|角色|
|:--:|:--:|:--:|
|192.168.88.111|6379|Redis Master|
|192.168.88.112|6379|Redis Slave|
|192.168.88.113|6379|Redis Slave|
|192.168.88.114|26379|Sentinel|
|192.168.88.115|26379|Sentinel|
|192.168.88.116|26379|Sentinel|

## 2. Redis安装与配置

本文使用的Redis版本为`redis-4.0.11`。

### 2.1 数据节点和哨兵节点通用安装

源码安装包redis-4.0.11.tar.gz放在`/opt`目录下。

```
# 解压
cd /opt
tar -zxvf redis-4.0.11.tar.gz
cd redis-4.0.11

# 安装
make
make install

# 创建相关文件夹
mkdir /usr/local/redis

# 拷贝启动命令文件
cd /opt/redis-4.0.11/src
cp redis-cli /usr/local/redis
cp redis-server /usr/local/redis
```

### 2.2 主从数据节点配置

拷贝配置文件到相关目录：
```
cd /opt/redis-4.0.11
cp redis.conf /usr/local/redis
```

编辑配置文件：
```
cd /usr/local/redis
vim redis.conf
```
常用配置如下：
```
# 注释以下内容开启远程访问
# bind 127.0.0.1

# Redis使用后台模式
daemonize yes

# 关闭保护模式
protected-mode no

# 修改pidfile指向路径
pidfile  /usr/local/redis/redis.pid

#日志文件
logfile " /usr/local/redis/log/redis.log"

#数据库文件名
dbfilename dump.rdb

#数据库文存放目录
dir /usr/local/redis/data
```

如果是从节点服务器，则redis.conf配置文件中还需要添加如下内容：
```xml
# 192.168.88.111为主节点ip，6379为端口
slaveof 192.168.88.111 6379
```

启动服务：
```
cd /usr/local/redis
redis-server redis.conf
```

启动客户端：
```
redis-cli
redis-cli -p 6379
```

查看节点状态：
```
info replication
```

如果需要远程连接redis服务器，需要修改防火墙配置：
```
# centos 6
vim /etc/sysconfig/iptables
# 添加如下配置
-A INPUT -m state --state NEW -m tcp -p tcp --dport 6379 -j ACCEPT
service iptables restart

# centos 7
firewall-cmd --permanent --add-port=6379/tcp
firewall-cmd --reload
```

**没有使用哨兵模式，主redis服务器挂掉之后，从redis服务器只能读不能写。**

### 2.3 sentinel哨兵节点配置

拷贝sentinel配置文件到相关目录：
```
cd /opt/redis-4.0.11
cp sentinel.conf /usr/local/redis
```

编辑配置文件：
```
# 关闭保护模式
protected-mode no

# sentinel monitor [master-group-name] [ip] [port] [quorum]
# 该行的意思是：监控的master的名字叫做mymaster （可以自定义），地址为192.168.88.111:6379，行尾最后的一个2代表在sentinel集群中，多少个sentinel认为master死了，才能真正认为该master不可用了。
sentinel monitor mymaster 192.168.88.111 6379 2
```

启动sentinel：
```
cd /usr/local/redis
redis-server sentinel.conf --sentinel
```

防火墙设置：
```
vim /etc/sysconfig/iptables
# 添加如下配置
-A INPUT -m state --state NEW -m tcp -p tcp --dport 26379 -j ACCEPT
service iptables restart

# centos 7
firewall-cmd --permanent --add-port=26379/tcp
firewall-cmd --reload
```

## 3. 总结

Redis哨兵为Redis提供了高可用性。实际上这意味着你可以使用哨兵模式创建一个可以不用人为干预而应对各种故障的Redis部署。

哨兵模式还提供了其他的附加功能，如监控，通知，为客户端提供配置。

下面是在宏观层面上哨兵模式的功能列表：
**监控：**哨兵不断的检查master和slave是否正常的运行。
**通知：**当监控的某台Redis实例发生问题时，可以通过API通知系统管理员和其他的应用程序。
**自动故障转移：**如果一个master不正常运行了，哨兵可以启动一个故障转移进程，将一个slave升级成为master，其他的slave被重新配置使用新的master，并且应用程序使用Redis服务端通知的新地址。
**配置提供者：**哨兵作为Redis客户端发现的权威来源：客户端连接到哨兵请求当前可靠的master的地址。如果发生故障，哨兵将报告新地址。

**哨兵的分布式特性**
Redis哨兵是一个分布式系统：
哨兵自身被设计成和多个哨兵进程一起合作运行。有多个哨兵进程合作的好处有：
当多个哨兵对一个master不再可用达成一致时执行故障检测。这会降低错误判断的概率。
即使在不是所有的哨兵都工作时哨兵也会工作，使系统健壮的抵抗故障。毕竟在故障系统里单点故障没有什么意义。

Redis的哨兵、Redis实例(master和slave)、和客户端是一个有特种功能的大型分布式系统。

当然，Redis Sentinel 仅仅解决了 高可用 的问题，对于 **主节点** 单点写入和单节点无法扩容等问题，还需要引入 **Redis Cluster 集群模式** 予以解决。
