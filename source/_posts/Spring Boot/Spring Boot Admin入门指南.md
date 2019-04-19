---
title: Spring Boot Admin入门指南
date: 2019-04-18
categories: Spring Boot
---

## Spring Boot Actuator
Actuator是Spring Boot的模块，它在应用中添加了REST/JMS端点，方便监控和管理应用。端点提供了健康检查、指标监控、访问日志、线程转储、堆转储和环境信息等等。

## Spring Boot Admin
Actuator功能强大，便于其他应用使用端点（只需要简单的REST调用）。但是开发人员使用时就没那么方便了。对于开发人员，有良好的交互界面会更方便浏览监控数据和管理应用。这正是Spring Boot Admin做的工作。它为actuator端点提供了良好的交互界面，并提供了额外的特性。

Spring Boot Admin不是Spring团队提供的模块，它是由[Codecentric](https://blog.codecentric.de/en/)公司创建的，代码在[Github](https://github.com/codecentric/spring-boot-admin)上公开。

## Client And Server
不像Actuator，Spring Boot Admin由两部分组成：Client和Server。

Server部分包括Admin用户界面并独立运行于被监控应用。Client部分是包含在被监控应用中，并注册到Admin Server。

这样，即使应用挂掉了或者不能正常运行，监控的Server依然正常运行。假如你有多个应用（比如Spring Boot微服务应用），每个应用运行多个实例。对于传统的Actuator监控，很难单独访问每个应用，因为你要跟踪有多少实例及它们在哪运行。

对于Spring Boot Admin，被监控应用的每个实例（Client）在启动时注册到Server，每个实例在Admin Server就有一个单点，就可以检查它们的状态了。
![Spring Boot Admin](https://upload-images.jianshu.io/upload_images/292448-8fd9e6ea8a1709a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Server配置
首先对Spring Boot Admin Server进行配置。

创建一个Spring Boot工程，可以使用[Spring Initializr](https://start.spring.io/)创建。保证包含`web`模块。
创建工程后，第一件事就是添加Spring Boot Admin Server依赖：
```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.1.0</version>
</dependency>
```
> 注意：尽管Spring Boot Admin不是由Pivotal官方开发的，但是你会发现使用Spring Initializr创建工程时可以选择Spring Boot Admin的Client和Server模块。

接着需要在启动类中加入注解`@EnableAdminServer`来开启Admin Server。
```java
@SpringBootApplication
@EnableAdminServerpublic class SpringBootAdminServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootAdminServerApplication.class, args);
    }
}
```
现在运行程序并在浏览器打开`http://localhost:8080/`，可以看到如下界面：
![Admin Server](https://upload-images.jianshu.io/upload_images/292448-2d3bb1c6a118fe1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Server运行正常，但是没有Client注册。

## Client配置
和Server一样，第一步是向新建Client工程添加依赖：
```
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.1.0</version>
</dependency>
```
然后指定运行的Admin Server的url，即在`application.properties`中添加：
```
spring.boot.admin.client.url=http://localhost:8080
```

### 添加Actuator

现在可以同时运行Client和Server应用。只要保证没有端口冲突，因为两个应用默认端口都是8080。测试情况下，可以在`application.properties`中设置`server.port=0`，这样Client会使用一个随机端口启动。这样就可以测试使用不同的端口启动多个实例。

打开Admin Server界面就可以看到Client应用了。点击应用名称显示应用详情页。

![应用详情页](https://upload-images.jianshu.io/upload_images/292448-e39086793a9d0a69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果你看到上面最少信息的界面，意味着你的项目中没有添加Actuator。记住，Spring Boot Admin使用Actuator端点监控应用运行状况。幸运的是，只需要在项目中添加依赖，自动配置会解决其他问题。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

但是，大部分的端点默认是不对外暴露的，所以需要在`application.properties`添加配置使它们暴露：
```
management.endpoints.web.exposure.include=*
```

暴露Actuator端点后就可以在Admin Server上看到更多的信息了。
![image.png](https://upload-images.jianshu.io/upload_images/292448-64cf77cd77e14cf5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

更多细节可以查看[这篇文章](https://www.vojtechruzicka.com/spring-boot-actuator/)。

## 安全

现在所有服务都能正常运行，但是我们要保证Actuator端点和Admin管理界面的安全性。

### Client安全

如果你已经使用了Spring Security，上面的内容不会起作用，因为Actuator端点默认是被保护的，Admin Server不能访问它们。如果没有使用Spring Security，首先需要添加依赖：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
为了方便测试，可以配置`management.security.enabled=false`临时禁用Actuator端点的保护。如果使用基本身份认证，需要在配置文件中提供用户名和密码。Admin Server使用这些凭证来认证Client的Actuator端点。
```
spring.boot.admin.client.instance.metadata.user.name=client
spring.boot.admin.client.instance.metadata.user.password=client
```
默认情况下，如果没有配置Spring Boot使用默认用户`user`并在应用启动时自动生成密码。启动时你可以在控制台找到密码。如果你要在应用中明确指定用户名和密码，可以在配置文件中配置：
```
spring.security.user.name=client
spring.security.user.password=client
```

### Server安全

和Client一样，在Admin Server添加依赖：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
在`application.properties`配置登录Admin Server的用户名和密码：
```
spring.security.user.name=server
spring.security.user.password=server
```

然后在**Client**端也要添加这些凭证，否则不能注册到server。
```
spring.boot.admin.client.username=server
spring.boot.admin.client.password=server
```
回到**Server**模块，最后一件事是添加Spring Security配置来保证Admin管理界面的安全性。
```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        SavedRequestAwareAuthenticationSuccessHandler successHandler 
            = new SavedRequestAwareAuthenticationSuccessHandler();
        successHandler.setTargetUrlParameter("redirectTo");
        successHandler.setDefaultTargetUrl("/");

        http.authorizeRequests()
            .antMatchers("/assets/**").permitAll()
            .antMatchers("/login").permitAll()
            .anyRequest().authenticated().and()
            .formLogin().loginPage("/login")
            .successHandler(successHandler).and()
            .logout().logoutUrl("/logout").and()
            .httpBasic().and()
            .csrf()
            .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            .ignoringAntMatchers(
                "/instances",
                "/actuator/**"
             );
    }
}
```

这段代码的作用是：限制只有通过HTTP基本认证和登录用户可以使用Admin管理界面。登录界面和静态资源（javascript, HTML, CSS）是公开的，否则无法登录。它是基于cookie的CSRF保护。你可以看到在CSRF保护中有些路径被忽略了，因为Admin Server[缺少适当的支持](http://codecentric.github.io/spring-boot-admin/current/#_csrf_on_actuator_endpoints)。

重启Server，可以看到更加美观的登录界面。
![Spring Boot Admin Server登录界面](https://upload-images.jianshu.io/upload_images/292448-8d42699dcfe8a688.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 云服务发现

Spring Boot Admin client并不是唯一可以注册应用到Server的方式。Admin Server也支持Spring Cloud Service Discovery。你可以通过[官方文档](http://codecentric.github.io/spring-boot-admin/current/#spring-cloud-discovery-support)或[Spring Cloud Discovery with Spring Boot Admin](https://zoltanaltfatter.com/2018/05/15/spring-cloud-discovery-with-spring-boot-admin/)查看更多信息。

## 通知

服务监控后，如果服务出错当然希望能够得到通知。Spring Boot Admin提供了多种通知选项。

第一次访问Admin Server页面时，它会提示要求显示推送通知的权限。一旦出现问题，你会收到提示信息。
![通知](https://upload-images.jianshu.io/upload_images/292448-ec1562c77afcfb9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其他通知需要简单配置。一般只需要在`application.properties`配置就行。目前支持的服务有：
* Mail
* [Slack](https://codecentric.github.io/spring-boot-admin/current/#slack-notifications)
* [HipChat](https://codecentric.github.io/spring-boot-admin/current/#hipchat-notifications)
* [PagerDuty](https://codecentric.github.io/spring-boot-admin/current/#pagerduty-notifications)
* [OpsGenie](https://codecentric.github.io/spring-boot-admin/current/#opsgenie-notifications)
* [Let's Chat](https://codecentric.github.io/spring-boot-admin/current/#letschat-notifications)
* [Telegram](https://codecentric.github.io/spring-boot-admin/current/#ms-teams-notifications)
* [Microsoft Team](https://codecentric.github.io/spring-boot-admin/current/#telegram-notifications)

## 配置Email通知
如果要使用Email通知，需要在`Server`端添加Spring email依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```
然后需要指定SMTP服务器，它用来发送email通知和证书。更新Admin Server的`application.properties`。
```
spring.mail.host=smtp.foo.com
spring.mail.username=smtp-server-user
spring.mail.password=smtp-server-password
```
然后指定发送者和接收者。
```
# 发送邮箱
spring.boot.admin.notify.mail.from="Spring Boot Admin <noreply@foo.com>"
# 接收者邮箱列表，以逗号分隔
spring.boot.admin.notify.mail.to=alice@foo.com,bob@bar.com
# 抄送者邮箱列表，以逗号分隔
spring.boot.admin.notify.mail.cc=joe@foo.com
```

## 结论
Spring Boot Admin为Actuator端点提供了漂亮的管理界面，而且它可以集中监控多个应用，这在云端和微服务中是非常有用的。但是要确保在使用过程中保证Server和Client的安全。
更多信息可以查看[官方文档](http://codecentric.github.io/spring-boot-admin/current/)。

**参考：**[https://www.vojtechruzicka.com/spring-boot-admin/](https://www.vojtechruzicka.com/spring-boot-admin/)
