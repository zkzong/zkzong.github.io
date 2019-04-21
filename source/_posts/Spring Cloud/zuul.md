---
title: Zuul
date: 2019-04-21
categories: Spring Cloud
---

默认情况下，Zuul会代理所有注册到Eureka Server的微服务，并且Zuul的路由规则如下：
http://ZUUL_HOST:ZUUL_PORT/微服务在Eureka上的serviceId/**会被转发到serviceId对应的微服务。

Zuul可以使用Ribbon达到负载均衡的效果。

Zuul整合了Hystrix。

路由配置详解：
1. 自定义指定微服务的访问路径
配置`zuul.routes.指定微服务的serviceId = 指定路径`即可。如：
```
zuul:
  routes:
    microservice-provider-user: /user/**
```
这样设置，microservice-provider-user微服务就会被映射到/user/**路径。
2. 忽略指定微服务
使用zuul.ignored-services配置需要忽略的服务，多个用逗号分隔，如：
```
zuul:
  ignored-services: microservice-provider-user,microservice-consumer-movie
```
这样就可让zuul忽略microservice-provider-user和microservice-consumer-movie微服务，只代理其他微服务。
3. 忽略所有微服务，只路由指定微服务。
4. 同时指定微服务的serviceId和对应路径。
5. 同时指定path和url
6. 同时指定path和url，并且不破坏zuul的hystrix、ribbon特性。
7. 使用正则表达式指定zuul的路由匹配规则
8. 路由前缀
9. 忽略某些路径
