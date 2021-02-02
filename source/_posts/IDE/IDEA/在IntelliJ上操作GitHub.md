---
title: 在IntelliJ上操作GitHub
date: 2020-12-31
categories: IDEA
---

IntelliJ IDEA集成了对GitHub的支持，使上传代码到GitHub和从GitHub下载代码更加方便快捷。

### 1. 分享代码到GitHub

1. 首先需要在IntelliJ配置Git，如果没有正确配置会出现如下错误：

![Git Error](http://img.blog.csdn.net/20150603162341754)

通过File->Settings打开设置面板进行设置，如图：

![Git Config](http://img.blog.csdn.net/20150603162711871)
2. 第一次上传代码到GitHub操作如下：

![Share](http://img.blog.csdn.net/20150603163020850)

其间需要输入用户名和密码。
3. 非第一次上传代码，需要像使用Git命令一样，遵循Add->Commit->Push的方式。如图：

![Add Commit](http://img.blog.csdn.net/20150603163808713)
![Push](http://img.blog.csdn.net/20150603163709539)

其中Add这一步可以省略，直接Commit->Push。

### 2. 从GitHub上clone代码

1. 首先选择File->New->Project from Version Control->GitHub

![GitHub](http://img.blog.csdn.net/20150603165333451)
2. 上步操作会打开如下界面：

![Clone](http://img.blog.csdn.net/20150603165351061)

在Git Repository URL输入需要clone的项目的url即可。
