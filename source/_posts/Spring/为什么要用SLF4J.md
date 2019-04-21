---
title: 为什么要用SLF4J
date: 2019-04-21
categories: Spring
---

## 阿里巴巴 Java 开发手册

前几天阿里巴巴在云栖社区首次公开阿里官方Java代码规范标准，就是一个PDF手册，有命名规范；有集合处理、并发处理、OOM/NPE 异常、魔法值（魔法值：即未经定义的常量）等等好多规范；还有一个关于 `Map 遍历`的推荐，这个大家应该都知道，推荐使用 entrySet 遍历 Map 类集合 KV，而不是 keySet 方式进行遍历。 因为 keySet 是遍历了 2 次，而 entrySet 只是遍历了一次就把 key 和 value 都放到了 entry 中，效率更高。还有接口类中的方法和属性不要加任何修饰符号（public 也不要加）这些推荐做法，这些都没什么，日常开发中应该做到的规范，但下面这个【强制】，我发现我接触的项目都没做到。

## 【强制】应用中不可直接使用日志系统（Log4j、Logback）中的 API

在手册中的日志规约中，看到有一条这样的规定，说实话我有点懵逼， Log4j 不是 Java 中应用最广的日志系统么？为啥不让用？

> 【强制】应用中不可直接使用日志系统（Log4j、Logback）中的API，而应依赖使用日志框架
> SLF4J中的API，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。
> ```
> import org.slf4j.Logger;
> import org.slf4j.LoggerFactory;
> private static final Logger logger = LoggerFactory.getLogger(Abc.class);
> ```

在这段规约中看到了推荐使用 **SLF4J** 这个日志框架，而且还是毫不由分说的【强制】，那它到底好在什么地方？

> SLF4J，即简单日志门面（Simple Logging Facade for Java），不是具体的日志解决方案，它只服务于各种各样的日志系统。按照官方的说法，SLF4J是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志系统。

大概意思就是说 SLF4J 是一个日志抽象层，允许你使用任何一个日志系统，并且可以随时切换还不需要动到已经写好的程序，这对于第三方组件的引入的不同日志系统来说几乎零学习成本了，况且它的优点不仅仅这一个而已，还有简洁的占位符的使用和日志级别的判断，众所周知的日志读写一定会影响系统的性能，但这些特性都是对系统性能友好的。
官网地址：[https://www.slf4j.org/](https://www.slf4j.org/)

## 测试一下

说了那么多，下面就建立一个Maven项目，并用 SLF4J 来结合 **JDK14、Simple、Logback、Log4j** 做日志系统，在上述几个日志系统间随意切换，而且**不修改一行代码**，甚至不用修改一个字符。

**1. 首先建立一个简单的 Java 项目**（Maven Project），目录结构如下：

![目录结构](http://upload-images.jianshu.io/upload_images/292448-6ae030a7f3424ed7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**2. 在 pom.xml 中增加 SLF4J API 依赖包**

使用的目前最新稳定版 1.7.22 的 SLF4J：
```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.22</version>
</dependency>
```

接着并在测试项目中的 `App.java` 中加入日志输出代码，代码如下：

```
package xyz.mafly.SLF4JTest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Hello world!
 *
*/
public class App {

    final Logger logger = LoggerFactory.getLogger(App.class);

    private void test() {
        logger.info("这是一条日志信息 - {}", "mafly");
    }

    public static void main(String[] args) {
        App app = new App();
        app.test();

        System.out.println("Hello World!");
    }
}
```

到这里，代码就写完了。以后无论在 Log4j 还是 Logback 日志系统切换，都不需要修改这里的代码！一个字符都不需要！！

**3. JDK14 日志系统**

`pom.xml` 中增加 slf4j-jdk14 依赖包：
```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jdk14</artifactId>
    <version>1.7.22</version>
</dependency>
```

运行程序，即可看到如下图输出：

![slf4j_jdk14](http://upload-images.jianshu.io/upload_images/292448-7f1c8a4e562b8d0e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**4. Simple 日志系统**

在 `pom.xml` 中注释掉 JDK14 包节点，增加 slf4j-simple 依赖包：
```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.22</version>
</dependency>
```

运行程序，即可看到如下图不同输出：

![slf4j_simple](http://upload-images.jianshu.io/upload_images/292448-fb752741037b6ea9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**5. Log4j 日志系统（最常用）**

依然是在 `pom.xml` 中注释掉 Simple 包节点，增加 slf4j-log4j12 依赖包：
```
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.22</version>
</dependency>
<dependency>
	<groupId>log4j</groupId>
	<artifactId>log4j</artifactId>
	<version>1.2.17</version>
</dependency>
```

Log4j 除了导入 jar 包后，还需要增加一下日志格式的配置文件，我新增了一个`log4j.properties`的日志配置文件，具体 Log4j 详细配置我之前在 [《log4j 项目中的详细配置》](http://blog.mayongfa.cn/128.html) 这篇博客中写过。运行程序，即可看到如下图输出（输出格式可自己配置）：

![slf4j_log4j](http://upload-images.jianshu.io/upload_images/292448-898e648cea010886.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**6. Logback 日志系统**

在 `pom.xml` 中注释掉 Log4j 包节点，增加 slf4j-logback 依赖包：
```
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.1.9</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.1.9</version>
</dependency>
```

运行程序，也可看到如下图日志输出：

![slf4j_logback](http://upload-images.jianshu.io/upload_images/292448-422cb16ce7a0d75a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 总结一下

看完阿里巴巴的这个开发手册，的确学到了一些新知识和规范，SLF4J 只是其中一个知识点而已。
说回 SLF4J 这个日志框架，在下一个开源项目或内部类库中都强烈推荐使用 SLF4J ，它的好处不言而喻，这也是阿里巴巴强制使用的原因所在。希望这篇文章对你的项目中日志系统有所帮助，任何一个任何编程语言的开发者，都应该重视日志的重要性和编码规范，对你、团队和未来阅读你代码的人都好。
