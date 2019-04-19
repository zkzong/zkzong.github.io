---
title: Thread Runnable Callable
date: 2019-04-19
categories: 多线程
---

Runnable 接口
Thread 类

Thread有start()方法，Runnable没有。

开发多线程以实现Runnable接口为主。

实现Runnable接口相比继承Thread类有如下好处：

1. 避免继承的局限，一个类可以继承多个接口。
2. 适合于资源的共享。


接口Callable与线程功能密不可分，但和Runnable的主要区别为：
1. Callable接口的call方法可以有返回值，而Runnable接口的run()方法没有返回值。
2. Callable接口的call方法可以声明抛出异常，而Runnable接口的run()方法不可以声明抛出异常。
执行完Callable接口中的任务后，返回值是通过Future接口进行获得的。
