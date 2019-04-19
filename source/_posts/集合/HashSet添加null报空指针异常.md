---
title: HashSet添加null报空指针异常
date: 2019-04-19
categories: 集合
---

HashSet添加null报空指针异常。
```java
public class TestSet {
    public static void main(String[] args) {
        Set<Integer> hashSet = new HashSet<Integer>();

        hashSet.add(2);
        hashSet.add(5);
        hashSet.add(1);
        hashSet.add(null);  // will throw null pointer
        hashSet.add(999);
        hashSet.add(10);
        hashSet.add(10);
        hashSet.add(11);
        hashSet.add(9);
        hashSet.add(10);
        hashSet.add(000);
        hashSet.add(999);
        hashSet.add(0);

        Iterator<Integer> it = hashSet.iterator();
        while(it.hasNext()){
            int i = it.next();
            System.out.print(i+" ");
        }
    }
}
```
Java集合类不能存储基本数据类型（如果要存储基本数据类型可以使用第三方API，如[Torve](http://trove.starlight-systems.com/)），所以当执行如下代码：
```java
hashSet.add(2);
hashSet.add(5);
```
实际上执行的是：
```java
hashSet.add(new Integer(2));
hashSet.add(new Integer(5));
```
向HashSet中添加null值并不是产生空指针异常的原因，HashSet中是可以添加null值的。NPE是因为在遍历set时需要把值拆箱为基本数据类型：
```java
while(it.hasNext()){
    int i = it.next();
    System.out.print(i+" ");
}
```
如果值为null，JVM试图把它拆箱为基本数据类型就会导致NPE。
装箱相当于执行`Integer.valueOf(100)`。
拆箱相当于执行`i.intValue()`。
此时相当于null调用intValue()方法，所以报NPE。
可以把代码修改为：
```java
while(it.hasNext()){
    final Integer i = it.next();
    System.out.print(i+" ");
}
```

[http://stackoverflow.com/questions/14774721/why-set-interface-does-not-allow-null-elements](http://stackoverflow.com/questions/14774721/why-set-interface-does-not-allow-null-elements)
