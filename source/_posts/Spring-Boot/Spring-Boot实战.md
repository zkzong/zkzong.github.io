# 第1章 入门

## 1.1 Spring风云再起

### 1.1.1 重新认识Spring

### 1.1.2 Spring Boot精要
四个核心：
- 自动配置：针对很多Spring应用程序常见的应用功能，Spring Boot能自动提供相关配置。
- 起步依赖：告诉Spring Boot需要什么功能，他就能引入需要的库。
- 命令行界面：这是Spring Boot的可选特性，借此你只需写代码就能完成完整的应用程序，无需传统项目构建。
- Actuator：让你能够深入运行中的Spring Boot应用程序，一探究竟。

### 1.1.3 Spring Boot不是什么
1. Spring Boot不是应用服务器，只是嵌入了Servlet容器。
2. Spring Boot没有实现诸如JPA或JMS之类的企业级Java规范，不过它自动配置了某个JPA实现的Bean，以此支持JPA。
3. Spring Boot没有引入任何形式的代码生成，而是利用了Spring 4的条件化配置特性，
以及Maven和Gradle提供的传递依赖解析，以此实现Spring应用程序上下文里的自动配置。

## 1.2 Spring Boot入门

### 1.2.1 安装Spring Boot CLI
Spring Boot CLI有好几种安装方式：
+ 用下载的分发包进行安装
+ 用Groovy Environment Manager进行安装
+ 通过OS X Homebrew进行安装
+ 使用MacPorts进行安装

### 1.2.2 使用Spring Initializr初始化Spring Boot项目
Spring Initializr有几种用法：
+ 通过Web界面使用
+ 通过Spring Tool Suite使用
+ 通过IntelliJ IDEA使用
+ 使用Spring Boot CLI使用

1. 使用Spring Initializr的Web界面
[http://start.spring.io](http://start.spring.io)
2. 在Spring Tool Suite里创建Spring Boot项目
3. 在IntelliJ IDEA里创建Spring Boot项目
4. 在Spring Boot CLI里使用Initializr
 
## 1.3 小结

# 第2章 开发第一个应用程序

## 2.1 运用Spring Boot

### 2.1.1 查看初始化的Spring Boot新项目
@SpringBootApplication将三个有用的注解组合在一起：
+ Spring的@Configuration
+ Spring的@ComponentScan
+ Spring Boot的@EnableAutoConfiguration

### 2.1.2 Spring Boot项目构建过程解析
Spring Boot为Gradle和Maven提供了构建插件，以便辅助构建Spring Boot项目。

要是选择用Maven来构建应用程序， Initializr会替你生成一个pom.xml文件，其中
使用了Spring Boot的Maven插件。
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>{springBootVersion}</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```
pom.xml里的<dependency>没有指定版本。

## 2.2 使用起步依赖
只在构建文件里指定这些功能，让构建过程自己搞明白我们要什么东西，这正是Spring Boot起步依赖的功能。

### 2.2.1 指定基于功能的依赖
Spring Boot通过提供众多起步依赖降低项目依赖的复杂度。起步依赖本质上是一个Maven项
目对象模型（Project Object Model， POM），定义了对其他库的传递依赖，这些东西加在一起即
支持某项功能。很多起步依赖的命名都暗示了它们提供的某种或某类功能。

我们并不需要指定版本号，起步依赖本身的版本是由正在使用的Spring Boot的版本来决定
的，而起步依赖则会决定它们引入的传递依赖的版本。

查看依赖树：
```
gradle dependencies
mvn dependency:tree
```

### 2.2.2 覆盖起步依赖引入的传递依赖

Gradle
```
compile("org.springframework.boot:spring-boot-starter-web") {
	exclude group: 'com.fasterxml.jackson.core'
}
```
Maven
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>com.fasterxml.jackson.core</groupId>
		</exclusion>
	</exclusions>
</dependency>
```

## 2.3 使用自动配置

### 2.3.1 专注于应用程序功能

### 2.3.2 运行应用程序

### 2.3.3 刚刚发生了什么
Spring条件化配置：实现Condition接口，覆盖它的matches()方法。

## 2.4 小结

起步依赖帮助你专注于应用程序需要的功能类型，而非提供该功能的具体版本。与此同时，自动配置把你从样板式的配置中解放了出来。这些配置在没有Spring Boot的Spring应用程序里非常常见。

# 第3章 自定义配置

两种影响自动配置的方式——使用显式配置进行覆盖和使用属性进行精细化配置

## 3.1 覆盖Spring Boot自动配置

### 3.1.1 保护应用程序
```
compile("org.springframework.boot:spring-boot-starter-security")

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
用户名：user，密码：在打印日志中

### 3.1.2 创建自定义的安全配置

### 3.1.3 掀开自动配置的神秘面纱

## 3.2 通过属性文件外置配置
Spring Boot自动配置的Bean提供了300多个用于微调的属性。

Spring Boot应用程序有多种设置途径。Spring Boot能从多种属性源获得属性，包括如下几处：
1. 命令行参数
2. java:comp/env里的JNDI属性
3. JVM系统属性
4. 操作系统环境变量
5. 随机生成的带random.*前缀的属性（在设置其他属性时，可以引用它们，比如${random.long}）
6. 应用程序以外的application.properties或者application.yml文件
7. 打包在应用程序内的application.properties或者application.yml文件
8. 通过@PropertySource标注的属性源
9. 默认属性

这个列表按照优先级排序

application.properties和application.yml文件能放在以下四个位置：
1. 外置，在相对于应用程序运行目录的/config子目录里
2. 外置，在应用程序运行的目录里
3. 内置，在config包内
4. 内置，在Classpath根目录

这个列表按照优先级排序

如果在同一优先级位置同时有application.properties和application.yml，那么==application. yml里的属性会覆盖application.properties里的属性==。（==写反了？==）

### 3.2.1 自动配置微调
禁用模板缓存
spring.thymeleaf.cache=false

配置嵌入式服务器
server.port=8080

HTTPS服务：

用JDK的keytool工具来创建一个密钥存储（keystore）
```
server.port = 8443
server.ssl.key-store = classpath:mykeys.jks
server.ssl.key-store-password = 123456
server.ssl.key-password = 123456
```

配置日志

配置数据源

### 3.2.2 应用程序Bean的配置外置

Spring Boot的属性解析器会自动把驼峰规则的属性和使用连字符或下划线的同名属性关联起来。

### 3.2.3 使用Profile进行配置

@Profile

设置spring.profiles.active属性就能激活Profile，任意设置配置属性的方式都能用于设置这个值。

## 3.3 定制应用程序错误页面

# 第4章 测试

Spring的SpringJUnit4ClassRunner可以在基于Junit的应用程序测试里加载Spring应用程序上下文。

1. 使用特定于Profile的属性文件
使用application.properties可以创建额外的属性文件，遵循application-{profile}.properties这种命名格式，就能提供特定于Profile的属性。
2. 使用多Profile YAML文件进行配置
如果使用YAML来配置属性，则可以遵循与配置文件相同的命名规范，即创建application-
{profile}.yml这样的YAML文件，并将与Profile无关的属性继续放在application.yml里。
但既然用了YAML，就可以把所有Profile的配置属性都放在一个application.yml文件里。
**使用一组三个连字符（---）作为分隔符。**

## 4.1 集成测试自动配置
在大多数情况下，为Spring Boot应用程序编写测试时应该用@SpringApplicationConfiguration代替@ContextConfiguration。


## 4.2 测试Web应用程序

两个方案：
+ Spring Mock MVC
+ Web集成测试

### 4.2.1 模拟Spring MVC

要在测试里设置Mock MVC，可以使用MockMvcBuilders，该类提供了两个静态方法。
+ standaloneSetup()：构建一个Mock MVC，提供一个或多个手工创建并配置的控制器。
+ webAppContextSetup()：使用Spring应用程序上下文来构建Mock MVC，该上下文里
可以包含一个或多个配置好的控制器。

两者的主要区别在于， standaloneSetup()希望你手工初始化并注入你要测试的控制器，
而webAppContextSetup()则基于一个WebApplicationContext的实例，通常由Spring加载。
前者同单元测试更加接近，你可能只想让它专注于单一控制器的测试，而后者让Spring加载控制
器及其依赖，以便进行完整的集成测试。

### 4.2.2 测试Web安全

经过身份验证的请求又该如何发起呢？ Spring Security提供了两个注解。
+ @WithMockUser：加载安全上下文，其中包含一个UserDetails，使用了给定的用户名、
密码和授权。
+ @WithUserDetails：根据给定的用户名查找UserDetails对象，加载安全上下文。

## 4.3 测试运行中应用程序

@WebIntegrationTest

### 4.3.1 用随机端口启动服务器

@SpringApplicationConfiguration和@WebIntegrationTest已经被@SpringBootTest替代。

### 4.3.2 使用Selenium测试HTML页面

`testCompile("org.seleniumhq.selenium:selenium-java:2.45.0")`

# 第5章 Spring Boot CLI

Spring Boot CLI施展了很多技能。
+ CLI可以利用Spring Boot的自动配置和起步依赖。
+ CLI可以检测到正在使用的特定类，自动解析合适的依赖库来支持那些类。
+ CLI知道多数常用类都在哪些包里，如果用到了这些类，它会把那些包加入Groovy的默认
包里。
+ 应用自动依赖解析和自动配置后， CLI可以检测到当前运行的是一个Web应用程序，并自
动引入嵌入式Web容器（默认是Tomcat）供应用程序使用。

# 第7章 深入Actuator

## 7.1 揭秘Actuator的端点

**Actuator的端点**

|HTTP方法| 路 径| 描 述|
|--|--|--|
|GET| /autoconfig| 提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过|
|GET| /configprops| 描述配置属性（包含默认值）如何注入Bean|
|GET| /beans| 描述应用程序上下文里全部的Bean，以及它们的关系|
|GET| /dump| 获取线程活动的快照|
|GET| /env| 获取全部环境属性|
|GET| /env/{name}| 根据名称获取特定的环境属性值|
|GET| /health| 报告应用程序的健康指标，这些值由HealthIndicator的实现类提供|
|GET| /info| 获取应用程序的定制信息，这些信息由info打头的属性提供|
|GET| /mappings| 描述全部的URI路径，以及它们和控制器（包含Actuator端点）的映射关系|
|GET| /metrics| 报告各种应用程序度量信息，比如内存用量和HTTP请求计数|
|GET| /metrics/{name}| 报告指定名称的应用程序度量值|
|POST| /shutdown| 关闭应用程序，要求endpoints.shutdown.enabled设置为true|
|GET| /trace| 提供基本的HTTP请求跟踪信息（时间戳、 HTTP头等）|

## 7.2 连接Actuator的远程shell

Gradle：
```
compile("org.springframework.boot:spring-boot-starter-remote-shell")
```

Maven：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-remote-shell</artifactId>
</dependency>
```

ssh user@localhost -p 2000

密码在日志中

# 第8章 部署Spring Boot应用程序

## 8.1 衡量多种部署方式
Spring Boot应用程序有多种构建和运行方式：
+ 在IDE中运行应用程序（涉及Spring ToolSuite或IntelliJ IDEA）。
+ 使用Maven的spring-boot:run或Gradle的bootRun，在命令行里运行。
+ 使用Maven或Gradle生成可运行的JAR文件，随后在命令行中运行。
+ 使用Spring Boot CLI在命令行中运行Groovy脚本。
+ 使用Spring Boot CLI来生成可运行的JAR文件，随后在命令行中运行。

## 8.2 部署到应用服务器

### 8.2.1 构建WAR文件

Gradle：
```
apply plugin: 'war'

war {
    baseName = 'test'
    version = '0.0.1-SNAPSHOT'
    archiveName = 'demo.war'
}
```
archiveName：打包后的包名

Maven：
`<packaging>war</packaging>`

要使用SpringBootServletInitializer，只需创建一个子类，覆盖configure()方法
来指定Spring配置类。

```java
public class TestServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Ch08DeployApplication.class);
    }
}
```

构建命令：

`gradle build`

`mvn package`

如果没有删除Application里的main()方法，构建过程生成的WAR文件仍可直接运行，一如可执行的JAR文件：

`java -jar readinglist-0.0.1-SNAPSHOT.war`

这样一来，同一个部署产物就能有两种部署方式了！

**如果war包放在外部容器中运行，需要使用SpringBootServletInitializer，如果通过java -jar运行则不需要。**

# Spring Boot开发者工具

```
compile "org.springframework.boot:spring-boot-devtools"
```

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

## 自动重启

IDEA不能自动重启的解决办法：
[http://blog.csdn.net/u012410733/article/details/54316548](http://blog.csdn.net/u012410733/article/details/54316548)

可以设置spring.devtools.restart.exclude属性来覆盖默认的重启排除目录。

如果想彻底关闭自动重启，可以将spring.devtools.restart.enabled设置
为false。

还可以设置一个触发文件，必须修改这个文件才能触发重启。只需设置spring.devtools.restart.triggerfile属性。

## 全局配置开发者工具

可以在主目录（home directory）里创建一个名为.spring-boot-devtools.
properties的文件。（请注意，文件名用“ .”开头。）

要是想覆盖这些配置，可以在每个项目的application.properties或application.yml文件里设置
特定于每个项目的属性。

## Spring Boot依赖

如果你正在使用的起步依赖没有覆盖到某个库，而你需要使用这个库，
那就得在Maven或Gradle的构建说明里显式地声明这个依赖。

请注意，在这两种情况下，都不需要指定版本号， Spring Boot的依赖管理会替你处理这个问
题的。但是，如果想覆盖Spring Boot选择的版本，你也可以显式地提供一个版本号。
