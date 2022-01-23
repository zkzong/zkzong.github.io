---
title: 如何把Spring Boot项目的war包部署到tomcat
date: 2019-04-18
categories: Spring Boot
---

Spring Boot默认提供内嵌的tomcat，所以打包直接生成jar包，用`java -jar`命令就可以启动。但是，有时候我们更希望一个tomcat来管理多个项目，这种情况下就需要项目是war格式的包而不是jar格式的包。

本文介绍如何把Spring Boot项目的war包部署到tomcat。

把Spring Boot的war包部署到tomcat，需要三步：
- 继承SpringBootServletInitializer
- 标记嵌入的servlet容器为`provided`
- 更新packaging为war

## 1. 继承SpringBootServletInitializer
修改已存在的`@SpringBootApplication`类，使其继承SpringBootServletInitializer。

### 1.1 典型的Spring Boot JAR部署（更新该文件以支持war包部署）
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DeployWarApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(DeployWarApplication.class, args);
    }

}
```

### 1.2 Spring Boot war包部署
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

@SpringBootApplication
public class DeployWarApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(DeployWarApplication.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(DeployWarApplication.class);
    }
}
```

如果创建了额外的DeployWarApplication类部署，确保高速Spring Boot启动哪个main类。
```
  <properties>
        <start-class>com.zkzong.NewSpringBootWebApplicationForWAR</start-class>
  </properties>
```

## 2. 标记内嵌的servlet容器为`provided`
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
```

## 3. 更新packaging为war
```
<packaging>war</packaging>
```

需要注意的是这样部署的request url需要在端口后加上项目的名字才能正常访问。spring-boot更加强大的一点就是：即便项目是以上配置，依然可以用内嵌的tomcat来调试，启动命令和以前没变，还是：`mvn spring-boot:run`。

如果需要在spring boot中加上request前缀，需要在`application.properties`中添加`server.contextPath=/prefix/`即可。其中`prefix`为前缀名。这个前缀会在war包中失效，取而代之的是war包名称，如果war包名称和prefix相同的话，那么调试环境和正式部署环境就是一个request地址了。

参考：
https://zhidao.baidu.com/question/1897916337731983300.html
https://www.mkyong.com/spring-boot/spring-boot-deploy-war-file-to-tomcat/