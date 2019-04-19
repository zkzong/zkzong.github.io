---
title: Spring Boot CommandLineRunner和ApplicationRunner
date: 2019-04-18
categories: Spring Boot
---

在spring boot应用中，我们可以在程序启动之前执行任何任务。为了达到这个目的，我们需要使用`CommandLineRunner`或`ApplicationRunner`接口创建bean，spring boot会自动监测到它们。这两个接口都有一个`run()`方法，在实现接口时需要覆盖该方法，并使用`@Component`注解使其成为bean。`CommandLineRunner`和`ApplicationRunner`的作用是相同的。不同之处在于`CommandLineRunner`接口的`run()`方法接收String数组作为参数，而`ApplicationRunner`接口的`run()`方法接收`ApplicationArguments`对象作为参数。当程序启动时，我们传给`main()`方法的参数可以被实现`CommandLineRunner`和`ApplicationRunner`接口的类的`run()`方法访问。我们可以创建多个实现`CommandLineRunner`和`ApplicationRunner`接口的类。为了使他们按一定顺序执行，可以使用`@Order`注解或实现`Ordered`接口。

`CommandLineRunner`和`ApplicationRunner`接口的`run()`方法在`SpringApplication`完成启动时执行。启动完成之后，应用开始运行。`CommandLineRunner`和`ApplicationRunner`的作用是在程序开始运行前执行任务或记录信息。

下面展示了如何在程序中使用`CommandLineRunner`和`ApplicationRunner`。

## Maven配置
**pom.xml**
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.concretepage</groupId>
	<artifactId>spring-boot-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>
	<name>spring-demo</name>
	<description>Spring Boot Demo Project</description>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.2.RELEASE</version>
	</parent>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
	    <dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter</artifactId>
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

## CommandLineRunner

`CommandLineRunner`是个接口，有一个`run()`方法。为了使用`CommandLineRunner`我们需要创建一个类实现该接口并覆盖`run()`方法。使用`@Component`注解实现类。当`SpringApplication.run()`启动spring boot程序时，启动完成之前，`CommandLineRunner.run()`会被执行。`CommandLineRunner`的`run()`方法接收启动服务时传过来的参数。
```
run(String... args)
```
参数为String数组。

**CommandLineRunnerBean.java**
```
package com.concretepage.bean;

import java.util.Arrays;
import java.util.stream.Collectors;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
public class CommandLineRunnerBean implements CommandLineRunner {
    private static final Logger logger = LoggerFactory.getLogger(CommandLineRunnerBean.class);	
    public void run(String... args) {
    	String strArgs = Arrays.stream(args).collect(Collectors.joining("|"));
    	logger.info("Application started with arguments:" + strArgs);
    }
} 
```

**MyApplication.java**
```
package com.concretepage;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import com.concretepage.service.HelloService;

@SpringBootApplication
public class MyApplication {
	private static final Logger logger = LoggerFactory.getLogger(MyApplication.class);
	public static void main(String[] args) {
		ConfigurableApplicationContext context = SpringApplication.run(MyApplication.class, args);
		HelloService service =  context.getBean(HelloService.class);
		logger.info(service.getMessage());
        }       
} 
```

我们也可以创建一个service。一旦Spring boot启动完成service就会执行。这意味着`SpringApplication.run()`执行完成后service的方法就会执行。
**HelloService.java**
```
package com.concretepage.service;

import org.springframework.stereotype.Service;

@Service
public class HelloService {
	public String getMessage(){
		return "Hello World!";
	}
}
```
现在使用带有参数的可执行jar运行程序。`spring-boot-demo-0.0.1-SNAPSHOT.jar`为生成的jar文件。执行命令如下：
```
java -jar spring-boot-demo-0.0.1-SNAPSHOT.jar data1 data2 data3 
```
输出结果为：
```
2017-03-19 13:38:38.909  INFO 1036 --- [           main] c.c.bean.CommandLineRunnerBean           : Application started with arguments:data1|data2|data3
2017-03-19 13:38:38.914  INFO 1036 --- [           main] com.concretepage.MyApplication           : Started MyApplication in 1.398 seconds (JVM running for 1.82)
2017-03-19 13:38:38.915  INFO 1036 --- [           main] com.concretepage.MyApplication           : Hello World! 
```

## ApplicationRunner

`ApplicationRunner`和`CommandLineRunner`的作用相同。在`SpringApplication.run()`完成spring boot启动之前，`ApplicationRunner`的`run()`方法会被执行。
```
run(ApplicationArguments args) 
```
可以看到，`CommandLineRunner.run()`接收String数组作为参数，而`ApplicationRunner.run()`接收`ApplicationArguments`作为参数。这些参数是启动spring boot程序时传给`main()`方法的。

**ApplicationRunnerBean.java**
```
package com.concretepage.bean;

import java.util.Arrays;
import java.util.stream.Collectors;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
public class ApplicationRunnerBean implements ApplicationRunner {
	private static final Logger logger = LoggerFactory.getLogger(ApplicationRunnerBean.class);	
	@Override
	public void run(ApplicationArguments arg0) throws Exception {
    	    String strArgs = Arrays.stream(arg0.getSourceArgs()).collect(Collectors.joining("|"));
    	    logger.info("Application started with arguments:" + strArgs);
	}
} 
```
创建可执行jar包并使用如下参数运行：
```
java -jar spring-boot-demo-0.0.1-SNAPSHOT.jar data1 data2 data3 
```
输出结果为：
```
2017-03-19 16:26:06.952  INFO 5004 --- [           main] c.c.bean.ApplicationRunnerBean           : Application started with arguments:data1|data2|data3
2017-03-19 16:26:06.956  INFO 5004 --- [           main] com.concretepage.MyApplication           : Started MyApplication in 1.334 seconds (JVM running for 1.797)
2017-03-19 16:26:06.957  INFO 5004 --- [           main] com.concretepage.MyApplication           : Hello World! 
```

## CommandLineRunner和ApplicationRunner的执行顺序

在spring boot程序中，我们可以使用不止一个实现`CommandLineRunner`和`ApplicationRunner`的bean。为了有序执行这些bean的`run()`方法，可以使用`@Order`注解或`Ordered`接口。例子中我们创建了两个实现`CommandLineRunner`接口的bean和两个实现`ApplicationRunner`接口的bean。我们使用`@Order`注解按顺序执行这四个bean。

**CommandLineRunnerBean1.java**
```
@Component
@Order(1)
public class CommandLineRunnerBean1 implements CommandLineRunner {
    public void run(String... args) {
    	System.out.println("CommandLineRunnerBean 1");
    }
}
```

**ApplicationRunnerBean1.java**
```
@Component
@Order(2)
public class ApplicationRunnerBean1 implements ApplicationRunner {
	@Override
	public void run(ApplicationArguments arg0) throws Exception {
		System.out.println("ApplicationRunnerBean 1");
	}
}
```

**CommandLineRunnerBean2.java**
```
@Component
@Order(3)
public class CommandLineRunnerBean2 implements CommandLineRunner {
    public void run(String... args) {
    	System.out.println("CommandLineRunnerBean 2");
    }
}
```

**ApplicationRunnerBean2.java**
```
@Component
@Order(4)
public class ApplicationRunnerBean2 implements ApplicationRunner {
	@Override
	public void run(ApplicationArguments arg0) throws Exception {
		System.out.println("ApplicationRunnerBean 2");
	}
} 
```
输出结果为：
```
CommandLineRunnerBean 1
ApplicationRunnerBean 1
CommandLineRunnerBean 2
ApplicationRunnerBean 2 
```
