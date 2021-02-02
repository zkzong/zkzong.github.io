---
title: MyEclipse web项目导入IntelliJ并部署运行
date: 2021-02-02
categories: eclipse
---

### 1. 导入MyEclipse项目
`File ｜ Project from Existing Sources... | 选择项目 | Import project from external model | Eclipse`
或
`File | Open`
### 2. 配置artifact
`File | Project Structure | Artifacts | + | Web Application | From Modules`
### 3. 部署到Tomcat
1. `Run | Edit Configurations |  + | Tomcat Server  | Local or Remote`
2. `Server | Application server`
3. `Deployment | + | Artifact`

支持热部署，修改如下配置：
![IntelliJ Tomcat Configuration](http://leanote.com/api/file/getImage?fileId=569ef01dab64415be1006c7b)

参考文章：
[http://my.oschina.net/tsl0922/blog/94621](http://my.oschina.net/tsl0922/blog/94621)
[http://www.php-note.com/article/detail/854](http://www.php-note.com/article/detail/854)
