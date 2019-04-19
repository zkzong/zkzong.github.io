---
title: Jenkins
date: 2019-04-18
categories: Jenkins
---

首先从Jenkins官方网站[https://jenkins.io/](https://jenkins.io/)下载最新的war包。虽然Jenkins提供了Windows、Linux、OS X等各种安装程序，但是，这些安装程序都没有war包好使。只需要运行命令：
```
java -jar jenkins.war
```
Jenkins就启动成功了！它的war包自带Jetty服务器，剩下的工作我们全部在浏览器中进行。
第一次启动Jenkins时，出于安全考虑，Jenkins会自动生成一个随机的按照口令。注意控制台输出的口令，复制下来，然后在浏览器输入：
```
http://localhost:8080/
```
粘贴口令，进入安装界面，如果执行默认的安装，Jenkins就自动配置好了Maven、git等常用插件。如果插件安装失败，可以下载后离线安装。插件下载地址：[http://mirrors.jenkins-ci.org/plugins/](http://mirrors.jenkins-ci.org/plugins/)。最后，创建一个admin用户，完成安装。
用管理员账号登录Jenkins后，第一次使用前，需要在**“系统管理”->“Global Tool Configuration”->“Maven”**中新增一个Maven，直接输入一个名字，选中“自动安装”，Jenkins会自动下载并安装Maven：

![install-maven](http://upload-images.jianshu.io/upload_images/292448-aebe5ea35ebbc3d2?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后，在Jenkins首页选择“新建”，输入名字，选择“构建一个maven项目”：

![new-job](http://upload-images.jianshu.io/upload_images/292448-5a70c1790d3193e0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在配置页中，源码管理选择Git，填入地址：

![git-config](http://upload-images.jianshu.io/upload_images/292448-b657b0155e247d4b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

默认使用master分支。如果需要口令，在Credentials中添加用户名/口令，或者使用SSH Key。

构建触发器指定了触发一次构建的条件。推荐使用最简单的配置**“Poll SCM”**，它的意思是，定时检查版本库，发现有新的提交就触发构建。这种方式对git、SVN等所有版本管理系统都是通用的。

我们在日程表中填入：
```
* * * * *
```

![trigger](http://upload-images.jianshu.io/upload_images/292448-602074bf222795c1?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

表示每分钟检查一次。如果你觉得太频繁，可以改成“每3分钟检查一次”：
```
*/3 * * * *
```

在“Build”中，默认的Root POM是`pom.xml`。如果`pom.xml`不在根目录下，就填入子目录，例如：`wxapi/pom.xml`。

在Goals and options中，填入需要执行的mvn命令：`clean package`，Jenkins将执行如下命令：
```
mvn clean package
```
特殊参数也在这里填写，如`-DskipTests=true clean package`。
保存后，就可以执行自动化构建了。

点击一个构建任务，可以在Console Output中看到控制台详细输出，便于出错排查：

![console-output](http://upload-images.jianshu.io/upload_images/292448-c79765e998eec85d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 如何部署

如果要部署构建好的war包，可以在Post Steps中填上shell命令，直接用脚本部署。

另一种方式是创建另外一个构建项目，手动触发部署。

无论用哪种方式，都是为了确保编译、部署是通过CI服务器完成的，而不是某台开发机器。

### 如何创建Linux服务

有了Jenkins，我们就可以在内网或者租用一台EC2服务器来搭建CI环境，每月费用不到¥100。推荐Ubuntu Linux系统。因为我们不想每次登录到Linux去启动Jenkins，也不想写脚本来启动服务。推荐安装JDK后，配合supervisor，把Jenkins直接变成一个服务。

可以在Linux上创建一个`ci`用户，然后，用supervisor启动并指定9001端口：

```
# /etc/supervisor/conf.d/ci.conf

[program:ci]
command=java -jar /home/ci/jenkins.war --httpPort=9001
user=ci
autostart=true
autorestart=true
startsecs=30
startretries=5
```

Jenkins默认在当前用户的主目录下创建`.jenkins`目录，所有的配置文件、数据库都存放在里面，只需要备份这个目录就备份了整个CI配置。

这样，一个CI环境就搭建完毕。

集成本地tomcat：
https://blog.csdn.net/javahighness/article/details/52641694
