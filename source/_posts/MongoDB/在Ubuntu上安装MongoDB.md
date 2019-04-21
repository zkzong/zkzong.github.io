---
title: 在Ubuntu上安装MongoDB
date: 2019-04-21
categories: MongoDB
---

## 1. 简介

可以使用`.deb`包安装MongoDB社区版，也可以使用Ubuntu自己的MongoDB包。当然官方的MongoDB包会比较新。

`mongodb-org-server`包提供了初始化脚本，使用`/etc/mongod.conf`配置文件启动`mongod`。

包提供的默认`/etc/mongod.conf`配置文件把`bind_ip`设置为`127.0.0.1`。

## 2. 安装MongoDB

### 第一步 导入包管理系统使用的公钥

Ubuntu包管理工具（如dpkg或apt）使用GPG key保证了包的一致性和可靠性。使用如下命令导入GPG key：
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```

### 第二步 创建list文件

创建`/etc/apt/sources.list.d/mongodb-org-3.4.list`list文件
```
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```

### 第三步 reload本地包数据库

```
sudo apt-get update
```

### 第四步 安装MongoDB包

安装最新稳定版MongoDB
```
sudo apt-get install -y mongodb-org
```

## 3. 运行MongoDB

MongoDB默认把数据存储在`/var/lib/mongodb`，把日志文件存储在`/var/log/mongodb`。也可以在`/etc/mongod.conf`文件中指定日志和数据文件。

### 第一步 运行MongoDB

```
sudo service mongod start
```

### 第二步 验证MongoDB开启成功

检查`/var/log/mongodb/mongod.log`日志验证mongod是否启动成功
`[initandlisten] waiting for connections on port <port>`
`<port>`是在`/etc/mongod.conf`中配置的端口号，默认为27017。

### 第三步 停止MongoDB

```
sudo service mongod stop
```

### 第四步 重启MongoDB

```
sudo service mongod restart
```

### 第五步 开始使用MongoDB

## 4. 卸载MongoDB

为了完全删除MongoDB，必须删除MongoDB应用，配置文件以及数据和日志文件。

### 第一步 停止MongoDB服务

```
sudo service mongod stop
```

### 第二步 移除包

```
sudo apt-get perge mongodb-org*
```

### 第三步 删除数据

```
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb
```
