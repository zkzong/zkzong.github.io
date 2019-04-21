---
title: Spring Cloud微服务实战
date: 2019-04-21
categories: Spring Cloud
---

## Spring Boot
YAML目前还有—些不足，它无法通过@PropertySource注解来加载配置。

@Value注解加载属性值的时候可以支持两种表达式来进行配置，如下所示。

+ 一种是PlaceHolder方式，格式为＄｛．．．｝，大括号内为PlaceHolder。
+ 另一种是使用SpEL 表达式(Spring Expression Language)，格式为＃｛．．．｝，大括号内为SpEL表达式。

## Eureka

Spring Cloud Eureka，使用Netflix Eureka来实现服务注册与发现，它既包含了服务端组件，也包含了客户端组件。Eureka服务端，也称为服务注册中心。Eureka客户端，主要处理服务的注册与发现。

在默认设置下，服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需在application.properties中增加如下配置：
```
server.port=1111

eureka.instance.hostname=localhost

eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
```
+ eureka.client.register-with-eureka: 由于该应用为注册中心，所以设置为false, 代表不向注册中心注册自己。
+ eureka.client.fetch-registry: 由于注册中心的职责就是维护服务实例，它并不需要去检索服务， 所以也设置为false。

Ribbon是一个基于HTTP和TCP的客户端负载均衡器，它可以在通过客户端中配置的ribbonServerList服务端列表去轮询访问以达到均衡负载的作用。当Ribbon与Eureka联合使用时，Ribbon的服务实例清单RibbonServerList会被DiscoveryEnabledNIWSServerList重
写，扩展成从Eureka注册中心中获取服务端列表。同时它也会用NIWSDiscoveryPing来取代IPing，它将职责委托给Eureka来确定服务端是否已经启动。

为了服务注册中心的安全考虑，很多时候我们都会为服务注册中心加入安全校验。这个时候，在配置serviceUrl时，需要在value值的URL中加入相应的安全校验信息，比如http://<username>:<password>@localhost:1111/eureka。其中，<username>为安全校验信息的用户名，<password>为该用户的密码。

## Ribbon

Spring Cloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，不需要独立部署。

通过Spring Cloud Ribbon的封装，在微服务架构中使用客户端负载均衡调用非常简单，只需要如下两步：
+ 服务提供者只需要启动多个服务实例并注册到一个注册中心或是多个相关联的服务注册中心。
+ 服务消费者直接通过调用被@LoadBalanced注解修饰过的RestTemplate来实现面向服务的接口调用。

### GET请求

在RestTemplate中，对GET请求可以通过如下两个方法进行调用实现。
1. getForEntity函数
2. getForObject函数

### POST请求

在RestTemplate中，对POST请求可以通过如下三个方法进行调用实现。
1. postForEntity函数
2. postForObject函数
3. postForLocation函数

### PUT请求

在RestTemplate中，对PUT请求可以通过put方法进行调用实现。

### DELETE请求

在RestTemplate中，对DELETE请求可以通过delete方法进行调用实现。

Spring Cloud Ribbon默认实现了区域亲和策略，可以通过Eureka实例的元数据配置来实现区域化的实例配置方案：
eureka.instance.metadataMap.zone=shanghai

禁用Eureka对Ribbon服务实例的维护实现：
ribbon.eureka.enabled=false

## Hystrix

@SpringCloudApplication

## Feign

Spring Cloud Feign基于Netflix Feign实现，整合了Spring Cloud Ribbon和Spring Cloud Hystrix，除了提供这两者的强大功能之外，它还提供了一种声明式的Web服务客户端定义方式。

在定义各参数绑定时，@RequestParam、@RequestHeader 等可以指定参数名称的注解， 它们的 value 千万不能少。 在 SpringMVC 程序中，这些注解会根据参数名来作为默认值，但是在Feign 中绑定参数必须通过 value 属性来指明具体的参数名，不然会抛出口legalStateException 异常， value 属性不能为空。

### Ribbon配置

#### 全局配置

ribbon.ConnectTimeout=500
ribbon.ReadTimeout=SOOO

#### 指定服务配置

使用@FeignClient注解，在初始化过程中，Spring Cloud Feign会根据该注解的name属性或value属性指定的服务名，自动创建一个同名的Ribbon客户端。也就是说，在之前的示例中，使用@FeignClient(value = "HELLO-SERVICE")来创 建 Feign 客户端的时候，同时也创建了一个名为HELLO-SERV工CE的Ribbon客户端。既然如此，我们就可以使用@FeignClient注解中的name或value属性值来设置对应的Ribbon参数，比如：
```
HELLO-SERVICE.ribbon.ConnectTimeout=SOO
HELLO-SERVICE.ribbon.ReadTimeout=2000
HELLO-SERVICE.ribbon.OkToRetryOnAllOperations=true
HELLO-SERVICE.ribbon.MaxAutoRetriesNextServer=2
HELLO-SERVICE.ribbon.MaxAutoRetries=l
```

### Hystrix配置

#### 全局配置

hystrix.command.default.execution.isolation.thread.timeoutinMilliseconds=5OOO
feign.hystrix.enabled=false
hystrix.command.default.execution.timeout.enabled=false

#### 禁用Hystrix

如果不想全局地关闭Hystrix支持, 而只想
针对某个 服务客户端关闭Hystrix 支待时, 需要通过使用@Scope ("prototype")注解为
指定的客户端配置Feign.Builder实例, 详细实现步骤如下所示。

+ 构建一个关闭Hystrix的配置类
```
@Configuration
public class DisableHystrixConfiguration {
    @Bean
    @Scope("prototype")
    public Feign.Builder feignBuilder() {
        return Feign.builder();
    }
}
```

+ 在HelloService的@FeignClient注解中,通过configuration参数引入上面实现的配置。

```
@FeignClient(name = "HELLO-SERVICE", configuration = DisableHystrixConfiguration.class)
public interface HelloService {
    ...
}
```

#### 指定命令配置

#### 服务降级配置

+ 通过@HystrixCommand注解的fallback参数来指定具体的服务降级处理方法。
+ 为Feign客户端的定义接口编写一个具体的接口实现类。

### 其他配置

#### 请求压缩

```
feign.compression.request.enabled=true
feign.compression.response.enabled=true
feign.compression.request.enabled=true

feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```

#### 日志配置

`logging.level.com.didispace.web.HelloSevice=DEBUG`

只是添加了如上配置, 还无法实现对 DEBUG 日志的输出。 这时由于 Feign 客
户端默认的 Logger.Level 对象定义为 NONE 级别, 该级别不会记录任何 Feign 调用过程
中的信息, 所以我们需要调整它的级别, 针对全局的日志级别, 可以在应用主类中直接加
入 Logger.Level 的 Bean 创建, 具体如下:
```
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication
public class ConsumerApplication {
	@Bean
	Logger.Level feignLoggerLevel() {
		return Logger.Level.FULL;
	}

	public static void main(String[J args) {
		SpringApplication.run(ConsumerApplication.class, args);
	}
}
```

也可以通过实现配置类，然后在具体的Feign客户端来指定配置类以实现是否要调整不同的日志级别，比如下面的实现:
```
@Configuration
public class FullLogConfiguration {
	@Bean
	Logger.Level feignLoggerLevel() {
		return Logger.Level.FULL;
	}
}

@FeignClient(name = "HELLO-SERVICE", configuration = FullLogConfiguration.class)
public interface HelloService {
	...
}
```

对于 Feign 的 Logger 级别主要有下面 4 类, 可根据实际需要进行调整使用。

+ NONE: 不记录任何信息。
+ BASIC: 仅记录请求方法、URL以及响应状态码和执行时间。
+ HEADERS: 除了记录BASIC级别的信息之外, 还会记录请求和响应的头信息。
+ FULL: 记录所有请求与响应的明细, 包括头信息、 请求体、 元数据等。

## Zuul

1. 路由规则与服务实例的维护
2. 对于类似签名校验、登录校验在微服务架构中的冗余问题

### 请求路由

### 请求过滤

1. 继承ZuulFilter实现类
2. 在Application中创建Bean
```
@Bean
public AccessFilter accessFilter() {
    return new AccessFilter();
}
```

- 它作为系统的统一入口,屏蔽了系统内部各个微服务的细节。
- 它可以与服务治理框架结合,实现自动化的服务实例维护以及负载均衡的路由转发。
- 它可以实现接口权限校验与微服务业务逻辑的解耦。
- 通过服务网关中的过炖器,在各生命周期中去校验请求的内容,将原本在对外服务层做的校验前移,保证了微服务的无状态性,同时降低了微服务的测试难度,让服务本身更集中关注业务逻辑的处理。

Ant风格的路径表达式：

通配符 | 说明
---|---
? | 匹配任意单个字符
* | 匹配任意数量的字符
** | 匹配任意数量的字符，支持多级目录

### Hystrix和Ribbon支持

当使用path与url的映射关系来配置路由规则的时候，对于路由转发的请求不会采用HystrixCommand来包装，所以这类路由请求没有现成隔离和断路器的保护，并且也不会有负载均衡的能力。因此，在使用Zuul的时候尽量使用path和serviceId的组合来进行配置。

## Config

client端的配置文件必须是`bootstrap.properties`。

获取配置属性的方法有两种：
1. @Value
2. Environment

### 加密解密
1. 替换JCE
2. 相关端点
/encrypt/status：查看加密功能状态的端点。
/key：查看密钥的端点。
/encrypt：对请求的body内容进行加密的端点。
/decrypt：对请求的body内容进行解密的端点。

keytool -genkeypair -alias config-server -keyalg RSA -keystore config-server.keystore

### 获取远程配置

1. 通过向Config Server发送GET请求以直接的方式获取，可用下面的链接形式

+ 不带{label}分支信息，默认访问master分支，可使用：
    - /{application}-{profile}.yml
    - /{application}-{profile}.properties
+ 带{label}分支信息，可使用：
    - /{label}/{application}-{profile}.yml
    - /{application}/{profile}[/{label}]
    - /{label}/{application}-{profile}.properties

2. 通过客户端配置方式加载的内容如下所示。

+ spring.application.name：对应配置文件中的{apptication}内容。
+ spring.cloud.config.profile：对应配置文件中{profile} 内容。
+ spring.cloud.config.label：对应分支内容，如不配置，默认为master。

### 动态刷新配置

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

通过POST请求发送到http://localhost:7002/refresh。

## Bus

### Kafka

```
tar -xzf kafka_2.11-0.11.0.0.tgz
# 启动zk
bin/zookeeper-server-start.sh config/zookeeper.properties
# 启动Kafka
bin/kafka-server-start.sh config/server.properties
# 创建topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
# 查看topic
bin/kafka-topics.sh --list --zookeeper localhost:2181
# 生产者
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
# 消费者
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```
