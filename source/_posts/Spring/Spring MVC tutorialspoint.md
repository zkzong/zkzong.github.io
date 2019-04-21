---
title: Spring MVC tutorialspoint
date: 2019-04-21
categories: Spring
---

# 1. Spring MVC简介
Spring MVC提供model-view-controller架构和已有组件，可以开发灵活、松散耦合的web应用。MVC模式把应用分成不同的概念（输入逻辑、业务逻辑和UI逻辑），组件之间松散耦合。

  + **Model**封装应用数据，一般上由POJO组成。
  + **View**负责渲染模型数据，一般上它会生成HTML，然后由客户端浏览器解析。
  + **Controller**负责处理用户请求和响应，构建合适的模型，传给**view**进行渲染。
  
## 1.1 DispatcherServlet
Spring MVC框架通过DispatcherServlet处理所有的HTTP请求和响应。Spring MVC中DispatcherServlet的请求流程如下图：

![请求流程](http://upload-images.jianshu.io/upload_images/292448-c92eea1a7983d364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

DispatcherServlet对传入的HTTP请求的事件响应顺序如下：

  1. 接收到HTTP请求后，DispatcherServlet通过HandleMapping调用合适的Controller。
  2. Controller收到请求，根据GET或者POST的不同调用合适的service方法。service方法根据业务逻辑设置模型数据，把view名称返回给DispatcherServlet。
  3. DispatcherServlet通过ViewResolver选取定义的view对请求进行响应。
  4. 一旦view初始化，DispatcherServlet把模型数据传给view，最后在浏览器渲染。
  
上面提到的组件，如HandlerMapping、Controller和ViewResolver是WebApplicationContext的一部分，WebApplicationContext是ApplicationContext的扩展，为web应用添加了一些必要的特性。

## 1.2 必需配置
需要在web.xml文件中使用URL映射手动配置DispatcherServlet处理的映射请求。下面代码展示了HelloWeb DispatcherServlet的定义和映射。

```xml
<web-app id="WebApp_ID" version="2.4"
    xmlns="http://java.sun.com/xml/ns/j2ee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
    http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
 
    <display-name>Spring MVC Application</display-name>

   <servlet>
      <servlet-name>HelloWeb</servlet-name>
      <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <load-on-startup>1</load-on-startup>
   </servlet>

   <servlet-mapping>
      <servlet-name>HelloWeb</servlet-name>
      <url-pattern>*.jsp</url-pattern>
   </servlet-mapping>

</web-app>
```

web.xml文件放在web应用的WebContent/WEB-INF目录下。首先，在HelloWeb DispatcherServlet初始化时，框架会试图从名为[servlet-name]-servlet.xml的文件加载应用上下文，这个文件位于WebContent/WEB-INF目录下。本例中文件名为HelloWeb-servlet.xml。其次，<servlet-mapping>标签指明由那个DispatcherServlet处理什么URL。这里所有以.jsp结尾的HTTP请求都由HelloWeb DispatcherServlet处理。

如果不想使用像[servlet-name]-servlet.xml的默认名称和WebContent/WEB-INF的默认位置，可以在web.xml中添加ContextLoaderListener的servlet监听器，自定义文件名称和位置，如下：

```xml
<web-app...>

<!-------- DispatcherServlet definition goes here----->
....
<context-param>
   <param-name>contextConfigLocation</param-name>
   <param-value>/WEB-INF/HelloWeb-servlet.xml</param-value>
</context-param>

<listener>
   <listener-class>
      org.springframework.web.context.ContextLoaderListener
   </listener-class>
</listener>
</web-app>
```
或
```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

现在来看下HelloWeb-servlet.xml这个配置文件，它位于WebContent/WEB-INF目录下：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:context="http://www.springframework.org/schema/context"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
   http://www.springframework.org/schema/beans     
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/context 
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:component-scan base-package="com.tutorialspoint" />

   <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name="prefix" value="/WEB-INF/jsp/" />
      <property name="suffix" value=".jsp" />
   </bean>

</beans>
```

以下是HelloWeb-servlet.xml文件的重要知识点：

+ [servlet-name]-servlet.xml文件用来创建定义的bean，重写全局范围定义的相同名称的bean。
+ <context:component-scan...>标签用来激活Spring MVC注解扫描功能，它使用了像@Controller和@RequestMapping等注解。
+ *InternalResourceViewResolver*有定义的规则处理视图名称。像上面定义的规则，名为hello的逻辑视图是/WEB-INF/jsp/hello.jsp的代理。
下一节介绍怎样创建实际的组件，如Controller、Model和View等。

## 1.3 定义Controller
DispatcherServlet把请求委托给controller以执行特定的功能。**@Controller**注解表明指定的类作为controller。**@RequestMapping**注解用来把URL映射到某个类或特定的处理方法。
```java
@Controller
@RequestMapping("/hello")
public class HelloController{
 
   @RequestMapping(method = RequestMethod.GET)
   public String printHello(ModelMap model) {
      model.addAttribute("message", "Hello Spring MVC Framework!");
      return "hello";
   }

}
```
**@Controller**注解定义类作为Spring MVC控制器。第一个**@RequestMapping**表明这个控制器中所有的处理方法都是相对/hello路径的。注解**@RequestMapping(method = RequestMethod.GET)**用来声明*pringHello()*方法作为控制器默认的方法处理HTTP GET请求。可以定义另一个方法处理相同URL的POST请求。
可以在上面controller的*@RequestMapping*中添加额外的属性，如下：
```java
@Controller
public class HelloController{
 
   @RequestMapping(value = "/hello", method = RequestMethod.GET)
   public String printHello(ModelMap model) {
      model.addAttribute("message", "Hello Spring MVC Framework!");
      return "hello";
   }

}
```
**value**属性表明URL和哪个处理方法映射，**method**属性表明方法处理HTTP GET请求。以下是controller中重要的知识点：

+ 在方法中可以定义需要的业务逻辑。也可以按要求在方法中调用另一个方法。
+ 基于定义的业务逻辑，可以在方法中创建**model**。可以设定不同的model属性，这些属性可以被view访问以呈现最终结果。例子中创建了一个包含“message”属性的model。
+ 定义的方法可以返回一个包含view名称的字符串，用来渲染model。例子中返回“hello”作为逻辑视图名称。

## 1.4 创建JSP视图
Spring MVC支持多种类型的视图，包括JSP，HTML，PDF，Excel，XML，Velocity模板，XSLT，JSON，Atom，RSS和JasperReport等。但是通常我们使用JSTL来实现JSP模板。**hello**视图（/WEB-INF/hello/hello.jsp）如下：
```jsp
<html>
   <head>
   <title>Hello Spring MVC</title>
   </head>
   <body>
   <h2>${message}</h2>
   </body>
</html>
```
`${message}`是在Controller中设置的属性。在视图中可以有多个属性显示。

**EL表达式无法获取值的解决办法：**
修改web.xml
```
<?xml version="1.0" encoding="UTF-8"?>

<web-app id="mvc" version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
    http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
```

## 1.5 Spring Web MVC实例
### 1.5.1 简单实例
通过URL`http://localhost:8080/HelloWeb/hello`访问
[代码下载](http://pan.baidu.com/s/1c0roH12)
### 1.5.2 表单提交实例
通过URL`http://localhost:8080/HelloWeb1/student`访问
`return new ModelAndView("student", "command", new Student());`如果在JSP文件中使用<form:form>标签，spring要求一个包含"command"的对象。
**@ModelAttribute**注解参考文章[Using @ModelAttribute on a method argument](http://docs.spring.io/spring/docs/3.1.x/spring-framework-reference/html/mvc.html#mvc-ann-modelattrib-method-args)
[代码下载](http://pan.baidu.com/s/1o62zEbg)
### 1.5.3 重定向实例
通过URL`http://localhost:8080/HelloWeb2/index`访问
[代码下载](http://pan.baidu.com/s/1c0roH12)
### 1.5.4 静态页面实例
通过URL`http://localhost:8080/HelloWeb3/index`访问
**<mvc:resources.../>**标签用来映射静态页面。**mapping**属性必须是Ant模式来指定http请求的URL。**location**属性必须指定一个或多个有静态页面的可用资源目录位置，静态页面可以是images、 stylesheets、JavaScript和其他静态内容。多个资源位置可以使用逗号分隔的值列表。
[代码下载](http://pan.baidu.com/s/1eQvovHw)
### 1.5.5 异常处理实例
通过URL`http://localhost:8080/HelloWeb4/student`访问
[代码下载](http://pan.baidu.com/s/1mgnAEgS)

原文：[http://www.tutorialspoint.com/spring/spring_web_mvc_framework.htm](http://www.tutorialspoint.com/spring/spring_web_mvc_framework.htm)
