---
title: yield vs join
date: 2019-04-19
categories: 多线程
---

## join
方法join的作用是使所属的线程对象x正常执行run()方法中的任务，而使当前线程z进行无限期的阻塞，等待线程x销毁后再继续执行线程z后面的代码。
方法join具有使线程排队运行的作用，有些类似同步的运行效果。join与synchronized的区别是：join在内部使用wait()方法进行等待，而synchronized关键字使用的是“对象监视器”原理做为同步。

## yield
yield()方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU执行时间。
