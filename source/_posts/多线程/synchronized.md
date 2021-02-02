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

---

synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：
1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；
3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；
4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用的对象是这个类的所有对象。

在用synchronized修饰方法时要注意以下几点：
1. synchronized关键字不能继承。
2. 在定义接口方法时不能使用synchronized关键字。
3. 构造方法不能使用synchronized关键字，但可以使用synchronized代码块来进行同步。

A. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类，该类所有的对象同一把锁。
B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。
C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。