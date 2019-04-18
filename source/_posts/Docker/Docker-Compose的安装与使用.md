---
title: Docker-Compose的安装与使用
date: 2019-04-18
categories: Docker
---

## Docker Compose简介

Docker Compose是一种用于通过使用单个命令创建和启动Docker应用程序的工具。我们可以使用它来配置应用程序的服务。

它是开发，测试和升级环境的利器。

它提供以下命令来管理应用程序的整个生命周期：
- 启动，停止和重建服务
- 查看运行服务的状态
- 测试运行服务的日志输出
- 在服务商运行一次性命令

要实现Docker Compose，需要包括以下步骤：
- 将应用程序环境变量放在Docker文件中以公开访问。
- 在`docker-compose.yml`文件中提供和配置服务名称，以便它们可以在隔离的环境中一起运行。
- 运行`docker-compose up`将启动并运行整个应用程序。

## Docker Compose安装

使用以下命令安装Docker Compose：
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
上面链接下载速度较慢，可以使用国内的镜像，如DaoCloud：
```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

给`docker-compose`添加执行的权限：
```
sudo chmod +x /usr/local/bin/docker-compose
```
使用`docker-compose version`查看是否安装成功。

## Docker Compose使用

创建一个`docker-compose.yml`配置文件：
```
version: '3'
services:
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat
    ports:
      - 8080:8080
```
参数说明：
- version：指定脚本语法解释器版本
- services：要启动的服务列表
    - webapp：服务名称，可以随便起名，不重复即可
        - restart：启动方式，这里的`always`表示总是启动，即使服务器重启了也会立即启动服务
        - image：镜像的名称，默认从Docker Hub下载
        - container_name：容器名称，可以随便起名，不重复即可
        - ports：端口映射列表，左边为宿主机端口，右边为容器端口

运行：
```
docker-compose up

# 后台运行
docker-compose up -d

# 删除
docker-compose down
```

## Docker Compose命令

**前台运行**
```
docker-compose up
```

**后台运行**
```
docker-compose up -d
```

**启动**
```
docker-compose start
```

**停止**
```
docker-compose stop
```

**停止并移除容器**
```
docker-compose down
```

**查看日志**
```
docker-compose logs -f tomcat
```
