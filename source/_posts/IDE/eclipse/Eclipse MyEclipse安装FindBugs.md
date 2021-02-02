---
title: Eclipse MyEclipse安装FindBugs
date: 2021-02-02
categories: eclipse
---

FindBugs是一个静态分析工具，用来查找Java代码中存在的bugs。
## 1. 安装FindBugs
FindBugs有两种安装方式：
+ 在线安装（Eclipse建议使用此安装方式）
+ 离线安装：下载FingdBugs插件，放入plugins文件夹（MyEclipse建议使用此安装方式）

## 2. Eclipse在线安装FindBugs
1. 打开Help -> Install New Software，点击Add按钮，弹出如下对话框
![Add Repository](http://img.blog.csdn.net/20151106141307324)
Location的值：`http://findbugs.cs.umd.edu/eclipse`
按照提示安装，完成之后重启即可。

## 3. MyEclipse使用插件安装FindBugs
1. 首先下载FindBugs插件，本文提供一个下载链接：[http://downloads.sourceforge.net/project/findbugs/findbugs%20eclipse%20plugin/1.3.9/edu.umd.cs.findbugs.plugin.eclipse_1.3.9.20090821.zip?use_mirror=ncu](http://downloads.sourceforge.net/project/findbugs/findbugs%20eclipse%20plugin/1.3.9/edu.umd.cs.findbugs.plugin.eclipse_1.3.9.20090821.zip?use_mirror=ncu)，也可去[官网](http://findbugs.sourceforge.net/index.html)下载。
2. 解压下载的文件，获取如下文件夹
![文件夹](http://img.blog.csdn.net/20151106141343177)
3. 把该文件夹拷贝到MyEclipse安装路径的`Common/plugins`目录下。
4. 修改bundles.info文件，该文件位于`D:\MyEclipse\MyEclipse 10\configuration\org.eclipse.equinox.simpleconfigurator`目录下。在bundles.info最后一行添加`edu.umd.cs.findbugs.plugin.eclipse,1.3.9.20090821,file:/D:/MyEclipse/Common/plugins/edu.umd.cs.findbugs.plugin.eclipse_1.3.9.20090821/,4,false`
5. 完成之后重启即可。

## 4. 使用
1. 在需要查找bug的java文件、包或项目上点击右键，选择Find Bugs。
![Find Bugs](http://img.blog.csdn.net/20151106143346740)
2. 在Bug Explorer中查看相关的bug情况。
![Bug Explorer](http://img.blog.csdn.net/20151106143412750)
如果没有找到Bug Explorer可以通过Window -> Show View -> Other打开。

参考文章：
[http://chenzhou123520.iteye.com/blog/1313565](http://chenzhou123520.iteye.com/blog/1313565)
[http://luckykapok918.blog.163.com/blog/static/2058650432012101394245604/](http://luckykapok918.blog.163.com/blog/static/2058650432012101394245604/)
[http://www.cnblogs.com/kayfans/archive/2012/06/18/2554022.html](http://www.cnblogs.com/kayfans/archive/2012/06/18/2554022.html)
