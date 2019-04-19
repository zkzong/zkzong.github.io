---
title: 实践Druid作为Spring Boot工程的数据源添加SQL监控
date: 2019-04-18
categories: Spring Boot
---

在大型业务系统上线后，为了保证系统能够更好地持续稳定运行，及时发现各种故障（代码缺陷、SQL性能问题、服务器CPU/磁盘参数指标和各类业务异常等），因此需要针对系统开发各种监控功能。在微服务架构下的各类业务平台中，针对SQL进行监控，并根据业务的发展情况及时进行调优尤为重要。如果让中间件或者业务研发团队自己根据业务特征定制化开发一套SQL的监控系统，可能既费时费力，又不一定能够达到预定的结果。本文将介绍业界较为流行的Druid数据源连接池插件，并跟其他几款热门的数据源连接池进行对比分析，最后给出在Spring Boot工程中集成该数据源连接池的实践方法。

## 1. Druid数据库连接池介绍
Druid数据源连接池来源于阿里巴巴，是淘宝和支付宝专用数据库连接池。事实上，它不仅仅是一个数据库连接池，还包含一个ProxyDriver、一系列内置的JDBC组件库、一个 SQL Parser。支持所有JDBC兼容的数据库，包括Oracle、MySql、Derby、Postgresql、SQL Server、H2等等。Druid针对Oracle和MySql做了特别优化，比如Oracle的PSCache内存占用优化，MySql的ping检测优化。Druid提供了诸如MySql、Oracle、Postgresql、SQL-92等SQL语句的完美支持，是一个手写的高性能SQL Parser，支持Visitor模式，使得分析SQL的抽象语法树很方便。它执行简单SQL语句耗时在10微秒以内，对于复杂的SQL语句耗时也在30微秒左右。另外，通过Druid提供的SQL Parser可以在JDBC层面上拦截SQL并进行相应处理，比如说分库分表、SQL安全审计等。Druid也能防御SQL注入攻击，WallFilter就是通过Druid的SQL Parser分析语义实现的。

## 2. 与其他几种数据库连接池进行对比
通过下面的表格先来看下Druid与当前比较流行的其他几款数据源连接池的对比：

|功能|dbcp|druid|c3p0|tomcat-jdbc|HikariCP|
|:--:|:--:|:--:|:--:|:--:|:--:|
|支持PSCache|是|是|是|否|否|
|监控|jmx|jmx/log/http|jmx,log|jmx|jmx|
|扩展|弱|好|弱|弱|弱|
|sql拦截及解析|无|支持|无|无|无|
|代码|简单|中等|复杂|简单|简单|
|特点|依赖common-pool|阿里开源，功能全面|代码逻辑复杂，且不易维护||功能简单，起源于boneCP|
|连接池管理|LinkedBlockingDeque|数组|更新|FairBlockingQueue|threadlocal+CopyOnWriteArrayList|
从上面对比的表格中，可以看到druid功能最为全面，具备sql拦截等功能，其中统计数据较为全面，具有良好的扩展性。虽然在性能方面比HikariCP略差，但是综合其他方面来考虑在做技术选型的时候，可以选择Druid作为数据源连接池组件来用。

## 3. 动手在Spring-Boot工程中添加Druid实践

本文前面两节都是主要讲了理论，相对比较枯燥。下面这一节将从实践的角度，来一步一步向大家展示如何在Spring Boot工程中添加Druid连接池进行业务级的SQL监控。

**版本环境**
Spring Boot 1.4.1.RELEASE、Druid 1.0.12、JDK 1.8

**在工程中添加Druid的pom依赖**
因为阿里开源了Druid的数据源连接池源码，我们可以通过maven仓库可以获得jar包依赖。访问`http://mvnrepository.com/artifact/com.alibaba/druid`选择自己项目需要的版本（在本次集成中选择的是1.0.12），点击进入后复制maven内容到pom.xml内即可，如下所示：
```
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.12</version>
</dependency>
```
在自己工程中添加完以上Druid数据源连接池的依赖后，记得在Intellij中点击下"Enable Auto import"选项即可自动下载maven依赖的jar到本地.m2目录并构建到项目中。添加Druid至Spring Boot工程中就这么Easy，这么快捷。

**在Spring Boot工程中添加Druid配置**
在上面我们已经将Druid添加至项目中，接下来需要修改Spring Boot的application.yml配置文件，来添加Druid数据源连接池的支持，如下所示：
```
server.port=8888
# 数据库访问配置
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=gbk&zeroDateTimeBehavior=convertToNull&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
# 下面为连接池的补充设置，应用到上面所有数据源中
spring.datasource.initialSize=5
spring.datasource.minIdle=5
spring.datasource.maxActive=20
# 配置获取连接等待超时的时间
spring.datasource.maxWait=60000
# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
spring.datasource.timeBetweenEvictionRunsMillis=60000
# 配置一个连接在池中最小生存的时间，单位是毫秒
spring.datasource.minEvictableIdleTimeMillis=300000
spring.datasource.validationQuery=SELECT 1 FROM DUAL
spring.datasource.testWhileIdle=true
spring.datasource.testOnBorrow=false
spring.datasource.testOnReturn=false
# 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
spring.datasource.filters=stat,wall,log4j
spring.datasource.logSlowSql=true
```
需要说明的是，上面配置中的`filters：stat`表示已经可以使用监控过滤器，这时结合定义一个过滤器，我们就可以用其来监控SQL的执行情况。

**开启Druid的SQL监控功能**
在工程中开启监控功能后，可以在工程应用运行过程中，通过Druid数据源连接池自带SQL监控提供的多维度数据，分析出业务SQL执行的情况，从而可以调整和优化代码以及SQL，方便业务开发同事调优数据库的访问性能。

要达到开启SQL监控的效果，还需在Spring Boot工程中还实现Druid数据源连接池的Serlvet以及Filter，其Bean的初始化代码如下（下面给出两种配置方式）：

### 第一种方式@Confing注解的配置类：
```
/**
 * druid 配置.
 * <p>
 * 这样的方式不需要添加注解：@ServletComponentScan
 *
 * @author Administrator
 */
@Configuration
public class DruidConfiguration {

    /**
     * 注册一个StatViewServlet
     *
     * @return
     */
    @Bean
    public ServletRegistrationBean DruidStatViewServle2() {
        //org.springframework.boot.context.embedded.ServletRegistrationBean提供类的进行注册.
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        servletRegistrationBean.addUrlMappings("/druid/*");

        //添加初始化参数：initParams

        //白名单：
        servletRegistrationBean.addInitParameter("allow", "127.0.0.1");
        //IP黑名单 (存在共同时，deny优先于allow) : 如果满足deny的话提示:Sorry, you are not permitted to view this page.
        servletRegistrationBean.addInitParameter("deny", "192.168.1.73");
        //登录查看信息的账号密码.
        servletRegistrationBean.addInitParameter("loginUsername", "admin");
        servletRegistrationBean.addInitParameter("loginPassword", "admin");
        //是否能够重置数据.
        servletRegistrationBean.addInitParameter("resetEnable", "false");
        return servletRegistrationBean;
    }

    /**
     * 注册一个：filterRegistrationBean
     *
     * @return
     */
    @Bean
    public FilterRegistrationBean druidStatFilter2() {

        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());

        //添加过滤规则.
        filterRegistrationBean.addUrlPatterns("/*");

        //添加不需要忽略的格式信息.
        filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid2/*");
        return filterRegistrationBean;
    }

}
```

### 第二种方式基于注解的配置：
```
/**
 * druid数据源状态监控.
 *
 * @author Administrator
 */
@WebServlet(urlPatterns = "/druid/*",
        initParams = {
                @WebInitParam(name = "allow", value = "192.168.1.72,127.0.0.1"),// IP白名单(没有配置或者为空，则允许所有访问)
                @WebInitParam(name = "deny", value = "192.168.1.73"),// IP黑名单 (存在共同时，deny优先于allow)
                @WebInitParam(name = "loginUsername", value = "admin"),// 用户名
                @WebInitParam(name = "loginPassword", value = "admin"),// 密码
                @WebInitParam(name = "resetEnable", value = "false")// 禁用HTML页面上的“Reset All”功能
        }
)
public class DruidStatViewServlet extends StatViewServlet {
    private static final long serialVersionUID = 1L;

}
```
```
/**
 * druid过滤器.
 *
 * @author Administrator
 */
@WebFilter(filterName = "druidWebStatFilter", urlPatterns = "/*",
        initParams = {
                @WebInitParam(name = "exclusions", value = "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")//忽略资源
        }
)
public class DruidStatFilter extends WebStatFilter {

}
```
使用上面第二种方式的话，还需要在Spring Boot工程的启动类上添加注解：@ServletComponentScan，这样使Spring能够扫描到我们自己编写的servlet和filter。

**使用Druid进行SQL监控的效果**
我们已经配置完成了Druid的监控，在本地运行Spring Boot的Jar包，运行成功后即可访问Druid监控界面，默认访问地址为：`http://localhost:8080/druid/`，最终的效果图如下所示：

![Login.png](https://upload-images.jianshu.io/upload_images/292448-bfe36b4744fcaf70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到了我们成功的访问了Druid的监控页面，那么现在输入我们在Bean初始化时候设置的用户名、密码（admin/admin）登录监控平台，进入监控平台首页，如下所示：

![image.png](https://upload-images.jianshu.io/upload_images/292448-979b6eb079648f43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/292448-bf0d2568d7e812b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
有了Web UI我们就可以方便的从这个UI上看到该工程部署起来后数据源初始化配置以及业务级SQL的执行情况。

## 4. 总结
本文围绕Druid数据源连接池为主题，先简要地介绍了该连接池的功能，然后通过与业界几款较为流行的数据源连接池进行横向对比，分析出Druid连接池的特色和优势。最后通过实践，进一步向大家阐述如何在一个Spring Boot工程中添加Druid连接池进行业务SQL级别的监控。

