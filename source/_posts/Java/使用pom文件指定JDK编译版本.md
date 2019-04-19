---
title: 使用pom文件指定JDK编译版本
date: 2019-04-18
categories: Java
---

maven项目会用`maven-compiler-plugin`默认的JDK版本来进行编译，如果不指明版本就容易出现版本不匹配的问题，可能导致编译不通过的问题。

解决办法：在pom文件中配置maven-compiler-plugin插件。

**有两种方式：**
**方式一：**
properties标签添加：
```
<maven.compiler.target>1.8</maven.compiler.target>
<maven.compiler.source>1.8</maven.compiler.source>
```
**方式二：**
```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.6.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```
