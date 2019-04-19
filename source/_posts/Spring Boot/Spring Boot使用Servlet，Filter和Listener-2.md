---
title: Spring Boot使用Servlet，Filter和Listener-2
date: 2019-04-18
categories: Spring Boot
---

上一篇文章介绍了在Spring Boot应用中如何使用`@Bean`的方式注册Servlet，Filter和Listener。

本文介绍如何使用`@WebServlet`，`@WebFilter`和`@WebListener`注解类分别注册Servlet，Filter和Listener。

## 依赖jar包

编辑`pom.xml`文件，添加`spring-boot-starter-web`依赖。
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.boraji.tutorial.springboot</groupId>
  <artifactId>spring-boot-servlet-filter-example</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>Spring Boot - Servlet, Filter and  Listener example</name>
  <packaging>war</packaging>
  <properties>
    <java.version>1.8</java.version>
  </properties>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
  </parent>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

## 创建Servlet，Filter和Listener

### Servlet
```
import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/myServlet")
public class MyServlet extends HttpServlet {

  private static final long serialVersionUID = 1L;

  @Override
  protected void doGet(HttpServletRequest req, HttpServletResponse resp)
      throws ServletException, IOException {
    System.out.println("MyServlet's doGet() method is invoked.");
    doAction(req, resp);
  }
  @Override
  protected void doPost(HttpServletRequest req, HttpServletResponse resp)
      throws ServletException, IOException {
    System.out.println("MyServlet's doPost() method is invoked.");
    doAction(req, resp);
  }
  
  private void doAction(HttpServletRequest req, HttpServletResponse resp)
      throws ServletException, IOException {
    String name = req.getParameter("name");
    resp.setContentType("text/plain");
    resp.getWriter().write("Hello " + name + "!");
  }
  
}
```

### Filter
```
import java.io.IOException;
import java.util.Enumeration;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;

@WebFilter("/myServlet")
public class MyFilter implements Filter {

  @Override
  public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
      throws IOException, ServletException {
    
    System.out.println("MyFilter doFilter() is invoked.");
    Enumeration<String> params = req.getParameterNames();
    while (params.hasMoreElements()) {
      String param=params.nextElement();
      System.out.println("Parameter:"+param+"\tValue:"+req.getParameter(param));
    }
    chain.doFilter(req, res);
  }

  @Override
  public void init(FilterConfig config) throws ServletException {
    
  }

  @Override
  public void destroy() {

  }

}
```

### Listener
```
import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;

@WebListener
public class MyServletContextListener implements ServletContextListener {
  @Override
  public void contextInitialized(ServletContextEvent e) {
    System.out.println("MyServletContextListener Context Initialized");
  }

  @Override
  public void contextDestroyed(ServletContextEvent e) {
    System.out.println("MyServletContextListener Context Destroyed");
  }

}
```

## 注册Servlet，Filter和Listener

注册servlet组件（如Servlet，Filter和Listener），需要使用`@ServletComponentScan`注解配置类。`@ServletComponentScan`这个注解会扫描指定包中的servlet组件。

创建`SpringBootApp`类，并使用注解`@ServletComponentScan`和`@SpringBootApplication`。
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@SpringBootApplication
@ServletComponentScan(basePackages = { 
    "com.boraji.tutorial.springboot.servlet",
    "com.boraji.tutorial.springboot.filter", 
    "com.boraji.tutorial.springboot.listener"
   })
public class SpringBootApp {
  public static void main(String[] args) {
    SpringApplication.run(SpringBootApp.class, args);
  }

}
```

## 运行

运行`SpringBootApp.java`类。可以通过`Run -> Run as -> Java Application`或者使用`mvn spring-boot：run`命令运行spring boot应用。

启动成功后，在浏览器输入`http://localhost:8080/myServlet?name=Sunil%20Singh%20Bora`。
![浏览器显示](https://upload-images.jianshu.io/upload_images/292448-8720f2ac8a37aa30.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

