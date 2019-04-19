---
title: Github上怎样把新commits的代码合并到自己的fork上
date: 2019-04-18
categories: Git
---

## 命令行模式

步骤：
1. **配置上游项目地址。**即将你 fork 的项目的地址给配置到自己的项目上。比如我 fork 了一个项目，原项目是 wabish/fork-demo.git，我的项目就是 cobish/fork-demo.git。使用以下命令来配置。
```
git remote add upstream https://github.com/wabish/fork-demo.git
```
然后可以查看一下配置状况，很好，上游项目的地址已经被加进来了。
```
git remote -v
origin  git@github.com:cobish/fork-demo.git (fetch)
origin  git@github.com:cobish/fork-demo.git (push)
upstream    https://github.com/wabish/fork-demo.git (fetch)
upstream    https://github.com/wabish/fork-demo.git (push)
```
2. **获取上游项目更新。**使用 fetch 命令更新，fetch 后会被存储在一个本地分支 upstream/master 上。
```
git fetch upstream
```
3. **合并到本地分支。**切换到 master 分支，合并 upstream/master 分支。
```
git merge upstream/master
```
4. **提交推送。**根据自己情况提交推送自己项目的代码。
```
git push origin master
```
由于项目已经配置了上游项目的地址，所以如果 fork 的项目再次更新，重复步骤 2、3、4即可。

## 图解模式

![步骤1](https://upload-images.jianshu.io/upload_images/292448-db73aaa39aa4b644.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![步骤2](https://upload-images.jianshu.io/upload_images/292448-ea9bfcf672e6bf72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![步骤3](https://upload-images.jianshu.io/upload_images/292448-bbd42635dc8b8b2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![步骤4](https://upload-images.jianshu.io/upload_images/292448-21ddb9cae8f30c44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

[https://www.zhihu.com/question/20393785/answer/30725725](https://www.zhihu.com/question/20393785/answer/30725725)
