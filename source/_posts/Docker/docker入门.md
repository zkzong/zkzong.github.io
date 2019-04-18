---
title: docker入门
date: 2019-04-18
categories: Docker
---

## 1. Docker for Linux

本篇文章适用于Linux发行版，如Ubuntu。

for [Windows](http://docs.docker.com/windows/started/) and [Mac OS X](http://docs.docker.com/mac/started/)

通过这篇文章你将学会：

+ 安装Docker
+ 在容器中运行镜像
+ 在Docker Hub中浏览镜像
+ 创建自己的镜像并在容器中运行
+ 创建Docker Hub帐号和镜像仓库
+ push镜像到Docker Hub

## 2. 安装Docker

### 2.1 **使用`sudo`权限登录Ubuntu**

### 2.2 检查`wget`是否安装

    which wget
    
如果没有安装，需要安装

    sudo apt-get update
    sudo apt-get install wget

### 2.3 获取最新的Docker包

    $ wget -qO- https://get.docker.com/ | sh
    
### 2.4 检查`docker`是否正确安装

    docker run hello-world
    
## 3. 理解镜像和容器

上一节运行`docker run hello-world`命令。使用这个命令完成使用Docker的主要任务。命令分三部分：

![http://docs.docker.com/tutimg/container_explainer.png](http://docs.docker.com/tutimg/container_explainer.png)

## 4. 查找并运行whalesay镜像

可以从Docker Hub上查找镜像。

### 4.1 定位whalesay镜像

1. 打开浏览器进入DockerHub
![http://docs.docker.com/tutimg/browse_and_search.png](http://docs.docker.com/tutimg/browse_and_search.png)
2. 单击`Browse & Search`
3. 在搜索框中输入`whalesay`
![http://docs.docker.com/tutimg/image_found.png](http://docs.docker.com/tutimg/image_found.png)
4. 单击docker/whalesay镜像
![http://docs.docker.com/tutimg/whale_repo.png](http://docs.docker.com/tutimg/whale_repo.png)
每个镜像仓库包含镜像信息。信息中包含如镜像是哪种软件和怎么使用等信息。

### 4.2 在容器中运行whalesay

1. 光标定位在终端的`$`之后
2. 输入`docker run docker/whalesay cowsay boo`
3. 输入`docker images`查看本地所有镜像

当在容器中运行镜像时，Docker会下载镜像到电脑中。本地的镜像可以节省时间。只有Hub中的镜像源改变后Docker才会下载镜像。当然也可以删除镜像。

## 5. 构建自己的镜像

### 5.1 创建Dockerfile文件

1. 打开终端
2. 使用`mkdir mydockerbuild`创建目录
3. 进入mydockerbuild目录创建Dockerfile文件
4. 在Dockerfile文件中输入以下内容
    `FROM docker/whalesay:latest`
    `RUN apt-get -y update && apt-get install -y fortunes`
    `CMD /usr/games/fortune -a | cowsay`
5. 保存Dockerfile文件
6. 使用`docker build -t docker-whale .`构建镜像

## 6. 创建Docker Hub账号和资源

和GitHub账号的注册基本一致，在此不详细介绍。
![http://docs.docker.com/tutimg/add_repository.png](http://docs.docker.com/tutimg/add_repository.png)

## 7. Tag、push、pull镜像

本节将介绍如何tag和push自己的镜像到新建的资源库，并测试和pull镜像。

### 7.1 tag、push镜像

1. 使用`docker images`列出本地镜像
2. 找到`docker-whale`镜像的`IMAGE ID`
3. 使用`IMAGE ID`和`docker tag`命令tag`docker-whale`镜像
![http://docs.docker.com/tutimg/tagger.png](http://docs.docker.com/tutimg/tagger.png)
`$ docker tag 7d9495d03763 maryatdocker/docker-whale:latest`
4. 使用`docker push`命令把镜像push到资源库
`docker push maryatdocker/docker-whale`

### 7.2 pull镜像

1. 使用`docker rmi`删除`maryatdocker/docker-whale`和`docker-whale`镜像
可以使用ID或name删除
`$ docker rmi -f 7d9495d03763`
`$ docker rmi -f docker-whale`
2. 使用`docker pull`命令从资源库中pull镜像
`docker pull yourusername/docker-whale`
3. 运行镜像
`docker run maryatdocker/docker-whale`

### 常见问题：
1. 命令没有使用`sudo`
2. 错误
`FATA[0000] Post http:///var/run/docker.sock/v1.18/containers/create: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?`
*解决方法：*`/etc/init.d/docker restart`

参考文章：[http://docs.docker.com/linux/started/](http://docs.docker.com/linux/started/)
