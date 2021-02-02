---
title: eclipse提交代码至GitHub
date: 2021-02-02
categories: eclipse
---

作为一名程序员，自己在学习时经常需要写代码，但是由于换电脑或其他原因这些代码可能丢失，不方便以后的查看和复习。如果有一个版本服务器，不仅能把上传代码，在需要是可以随时下载，而且能实现版本控制，查看每个版本做了哪些修改。这时GitHub是个不错的选择。
 1. 要使用GitHub首先需要注册一个GitHub账号，并创建一个Repository。这已基本成为每个程序员的必备技能，在此就不赘述了。
 2. 在eclipse上安装git插件
首先选择Help -> Install New Software：
![Install New Software](http://img.blog.csdn.net/20150922133020857)
弹出如下窗口，点击Add按钮：
![Add](http://img.blog.csdn.net/20150922133247112)
弹出如下窗口，输入相应内容：
![egit](http://img.blog.csdn.net/20150922133349409)
Name的值可以任意输入，建议见名知义；Location的值为`http://download.eclipse.org/egit/updates`。
往下选择默认的就ok了。安装完成之后需要重启eclipse。
需要在Window -> Preferences -> Team -> Git -> Configuration中配置GitHub的用户信息。
![Configuration](http://img.blog.csdn.net/20150922142433249)
3. 在eclipse中创建Java项目（本文以Java项目为例，其他项目与此类似）。在项目名字上右键选择Team -> Share Project：
![Share Project](http://img.blog.csdn.net/20150922121708142)
选择Git，点击下一步：
![Git](http://img.blog.csdn.net/20150922134432675)
第一次时需要勾选`Use or create repository in parent folder of project`
![Configure Git Repository](http://img.blog.csdn.net/20150922134636961)
选中项目，点击`Create Repository`
![Configure Git Repository](http://img.blog.csdn.net/20150922134820478)
完成后就在本机上创建了一个Git仓库。此时项目中文件会显示问号小图标。
![问号](http://img.blog.csdn.net/20150922135201108)
此时就可以把代码提交到本地仓库了，在项目上右键选择Team -> Commit
![Commit](http://img.blog.csdn.net/20150922135441523)
可以选择某个文件提交，也可以选择全部提交。`Commit message`为必填项。
![Commit Changes to Git Repository](http://img.blog.csdn.net/20150922135549384)
点击`Commit`按钮就可以把代码提交到本地仓库。当然也可以点击`Commit and Push`按钮提交代码到本地仓库并上传至GitHub。
如果点击的是Commit按钮，接下来就要把代码Push到GitHub上。右键项目选择Team -> Remote -> Push
![Push](http://img.blog.csdn.net/20150922140203482)
输入之前在GitHub上创建的Repository的URI
![Destination Git Repository](http://img.blog.csdn.net/20150922140353665)
Host和Repository path会自动生成，不需要输入。User和Password需要输入。
下一步选择分支，此处选择master而不是HEAD。
![Push Ref Specifications](http://img.blog.csdn.net/20150922140812493)
然后点击Add Spec
![Add Spec](http://img.blog.csdn.net/20150922140824327)
![这里写图片描述](http://img.blog.csdn.net/20150922140839127)
 点击完成，上传成功。此时在GitHub上查看代码是否已经上传。
 如果没有上传成功，可能是上一步没有勾选`Force Update`。建议每次上传都勾选。
![Force Update](http://img.blog.csdn.net/20150922140850860)
![OK](http://img.blog.csdn.net/20150922140904597)
至此全部完成。

## 从GitHub上clone代码
File -> Import -> Git -> Projects from Git -> Clone URI
