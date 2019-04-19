---
title: ThreadLocal
date: 2019-04-19
categories: 多线程
---

ThreadLocal使用场合主要解决多线程中数据因并发产生不一致的问题。

ThreadLocal类中一共有4个方法：
- T get()
- protected T initialValue()
- void remove()
- void set(T value)

ThreadLocal设置值有两种方案：
1. Override其initialValue方法
2. 通过set设置

对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，比如定义一个static变量，同步访问，而ThreadLocal采用了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。

**ThreadLocal建议：**
ThreadLocal类变量因为本身定位为要被多个线程来访问，它通常被定义为static变量。
能够通过值传递的参数，不要通过ThreadLocal存储，以免造成ThreadLocal的滥用。
在线程池的情况下，在ThreadLocal业务周期处理完成时，最好显式的调用remove()方法，清空“线程局部变量”中的值。
在正常情况下使用ThreadLocal不会造成OOM，弱引用的只是ThreadLocal，保存值依然是强引用，如果ThreadLocal依然被其他对象应用，线程局部变量将无法回收。


ThreadLocal类的作用是为每个线程都创建一个变量副本，每个线程都可以修改自己所拥有的变量副本，而不会影响其他线程的副本。其实这也是解决线程安全的问题的一种方法。
