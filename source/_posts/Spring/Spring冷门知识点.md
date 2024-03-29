---
title: Spring冷门知识点
date: 2019-04-21
categories: Spring
---

PathVariable类型的参数不需要在Model中明确设置就可以在jsp中获取。
```
@RequestMapping(value = "/{name}")
public String pathPage(@PathVariable String name) {
    return "path";
}
```
在jsp中可以直接通过`${name}`获取值。

post请求 @RequestParam 放在body
我们都知道get请求的参数都是放在url后面，如：http://localhost:8080/get?id=1，方法声明如下：`public String post1(@RequestParam String id)`，@RequestParam表明接受的是url后面的参数；而post请求的参数一般放在body里面，当然post请求的url后面也可以带参数，方法声明如下：`public String post(@RequestParam String id, @RequestBody User user)`，id就是url后面的参数，user就是body里面的参数。
但是@RequestParam类型的参数也是可以放在post请求的body中的。但是此时`Content-Type`需要是`application/x-www-form-urlencoded`。
```java
// Content-Type = application/x-www-form-urlencoded
@RequestMapping(value = "/post1", method = RequestMethod.POST)
@ResponseBody
public String post1(@RequestParam String id) {
    return "id=" + id;
}
```
body内容为`id=1`。
