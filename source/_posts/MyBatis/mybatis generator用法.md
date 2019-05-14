---
title: mybatis generator用法
date: 2019-05-14
categories: MyBatis
---

mybatis-generator有三种用法：命令行、eclipse插件、maven插件。个人推荐使用命令行或maven插件。

命令行的用法请参考：[http://blog.csdn.net/wyc_cs/article/details/9023117](http://blog.csdn.net/wyc_cs/article/details/9023117)
maven插件的用法请参考：[http://www.cnblogs.com/yjmyzz/p/mybatis-generator-tutorial.html](http://www.cnblogs.com/yjmyzz/p/mybatis-generator-tutorial.html)

## 下面重点介绍一下注意事项：

**实体类生成构造方法：**
`<javaModelGenerator>`中添加`<property name="constructorBased" value="true" />`

**实体类生成toString方法：**
`<plugin type="org.mybatis.generator.plugins.ToStringPlugin" />`
格式

**实体类实现序列化接口：**
`<plugin type="org.mybatis.generator.plugins.SerializablePlugin" />`
serialVersionUID值

**自增主键：**
自增主键在插入数据时一般不需要给这个主键赋值，可以使用以下配置：
`<table>`中添加`<generatedKey column="id" sqlStatement="JDBC" identity="true" />`

**mapper.xml追加而不是覆盖：**
生成的`.xml`文件在重新生成时是以追加的方式添加到xml文件中的，而不是覆盖，具体原因可以查看这篇文章[https://my.oschina.net/u/140938/blog/220006](https://my.oschina.net/u/140938/blog/220006)，因此每次都要删除xml文件再重新生成。

其他参考文章：
**plugins：**
http://www.mybatis.org/generator/reference/plugins.html

**Mybatis Generator配置详解：**
[http://www.jianshu.com/p/e09d2370b796](http://www.jianshu.com/p/e09d2370b796)
