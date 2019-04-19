---
title: BeanUtils.copyProperties()使用
date: 2019-04-18
categories: Java
---

大部分Java程序员应该都用过BeanUtils.copyProperties()，起作用就是把两个对象中相同字段进行赋值。
```
Person p1 = new Person("zong", 30);
Person p2 = new Person();
BeanUtils.copyProperties(p2, p1);
```
当然p2不一定也是Person对象，其他对象也可以。只要两个对象中有相同的成员变量就可以赋值。

***但是如果赋值对象是List等集合类呢？？？***
答案是否定的，它们之间并不能赋值。
```
Person p1 = new Person("zong", 30);
Person p2 = new Person("ma", 25);
Person p3 = new Person("liu", 20);
List<Person> sList = new ArrayList<Person>(3);
sList.add(p1);
sList.add(p2);
sList.add(p3);

List<Person> tList = new ArrayList<Person>(3);
BeanUtils.copyProperties(tList, sList);
System.out.println(tList.size());
```
以上代码是想把sList的值赋给tList，但是运行之后发现tList.size()的值为0，赋值失败。

如果要对两个List对象赋值，可以参考如下代码：
```
List<Person> tList = new ArrayList<Person>(3);
for (int i = 0; i < sList.size(); i++) {
	Person p = new Person();
	BeanUtils.copyProperties(p, sList.get(i));
	tList.add(p);
}
```

[http://stackoverflow.com/questions/19312055/beanutils-copyproperties-to-copy-arraylist](http://stackoverflow.com/questions/19312055/beanutils-copyproperties-to-copy-arraylist)
