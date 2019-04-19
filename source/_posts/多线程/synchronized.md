---
title: synchronized
date: 2019-04-19
categories: 多线程
---

Java语言的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。

1. 当两个并发线程访问同一个对象object中的这个synchronized(this)同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块。
2. 然而，当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该object中的**非synchronized(this)**同步代码块。
3. 尤其关键的是，当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对object中所有其它synchronized(this)同步代码块的访问将被阻塞。
4. 第三个例子同样适用其它同步代码块。也就是说，当一个线程访问object的一个synchronized(this)同步代码块时，它就获得了这个object的对象锁。结果，其它线程对该object对象所有同步代码部分的访问都被暂时阻塞。
5. 以上规则对其它对象锁同样适用。

synchronized关键字，它包括两种用法：**synchronized方法**和**synchronized块**。

**同步代码块和同步方法有小小的不同：**
从尺寸上讲，同步代码块比同步方法小。你可以把同步代码块看成是没上锁房间里的一块用带锁的屏风隔开的空间。
同步代码块还可以人为的指定获得某个其它对象的key。就像是指定用哪一把钥匙才能开这个屏风的锁，你可以用本房的钥匙；你也可以指定用另一个房子的钥匙才能开，这样的话，你要跑到另一栋房子那儿把那个钥匙拿来，并用那个房子的钥匙来打开这个房子的带锁的屏风。记住你获得的那另一栋房子的钥匙，并不影响其他人进入那栋房子没有锁的房间。

如果一个类中定义了一个synchronized的**静态方法A**，也定义了一个synchronized的**实例方法B**，那么这个类的同一对象Obj在多线程中分别访问A和B两个方法时，**不会构成同步**，因为它们的锁都不一样。**A方法的锁是Obj这个对象，而B的锁是Obj所属的那个Class。**

[http://www.cnblogs.com/GnagWang/archive/2011/02/27/1966606.html](http://www.cnblogs.com/GnagWang/archive/2011/02/27/1966606.html)

synchronized与static synchronized 的区别：
[http://www.cnblogs.com/shipengzhi/articles/2223100.html](http://www.cnblogs.com/shipengzhi/articles/2223100.html)
