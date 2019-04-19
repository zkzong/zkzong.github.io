---
title: List Set Map之间的不同
date: 2019-04-19
categories: 集合
---

它们都继承自**Collection**类。

**10点不同：**

|序号|属性|java.util.List|java.util.Set|java.util.Map|
|:--:|--|--|--|--|
|1|重复元素|List**允许**存储重复元素。|Set**不允许**存储重复元素。|Map以键值对形式存储数据，**key不允许重复，value可以重复。**|
|2|插入顺序|List以**插入顺序**存储元素。|**大部分**Set实现类不维护插入顺序。<br>HashSet不维护插入顺序。<br>**LinkedHashSet**维护插入顺序。<br>**TreeSet**以**自然顺序**排序。|**大部分**Map实现类不维护插入顺序。<br>HashMap不维护插入顺序。<br>**LinkedHashMap**维护key的插入顺序。<br>**TreeMap**以key的**自然顺序**排序。|
|3|null keys|List允许存储**多个null**值。|**大部分**Set实现类允许存储一个null值。<br>**TreeSet**和**ConcurrentSkipListSet** **不允许**存储null值。|Map实现类：<br>HashMap允许**一个null键**和**多个null值**。<br>LinkedHashMap允许**一个null键**和**多个null值**。<br>TreeMap**不允许null键**，**允许多个null值**。<br>Hashtable**不允许null键和null值**。<br>ConcurrentHashMap**不允许null键和null值**。<br>ConcurrentSkipListMap**不允许null键和null值**。|
|4|获取指定索引的元素|List实现类提供了get方法获取指定索引的元素。get方法直接通过指定索引获取元素，因此时间复杂度为O(1)。|Set实现类不提供此类方法。|Map实现类不提供此类方法。|
|5|子类|ArrayList<br>LinkedList<br>Vector<br>CopyOnWriteArrayList|HashSet<br>CopyOnWriteArraySet<br> LinkedHashSet<br>TreeSet<br>ConcurrentSkipListSet<br>EnumSet|HashMap<br>Hashtable<br> ConcurrentHashMap<br>LinkedHashMap<br>TreeMap<br>ConcurrentSkipListMap<br> IdentityHashMap<br>WeakHashMap<br>EnumMap|
|6|listIterator|listIterator方法遍历元素并返回ListIterator对象。<br>listIterator相对iterator方法提供了额外的方法：hasPrevious(), previous(), nextIndex(), previousIndex(), add(E element), set(E element)。|Set没有提供类似listIterator的方法，只是简单返回Iterator。|Map提供了三种iterator：<br>**map.keySet().iterator()**<br>遍历key并返回Iterator对象。<br>**map.values().iterator()**<br>遍历value并返回Iterator对象。<br>**map.entrySet().iterator()**<br>遍历key和value并返回Map.Entry对象。|
|7|结构和调整大小|List是可调整大小的数组。|Set使用Map实现。因此Set的结构和调整大小与Map相同。|Map使用哈希技术存储键值对。|
|8|基于结构/随机访问的索引|ArrayList使用基于索引的数组实现，因此提供了随机访问。LinkedList不是基于索引的结构。|Set不是基于索引的结构。|Map不是基于索引的结构。|
|9|非同步的子类|ArrayList<br>LinkedList|HashSet<br>LinkedHashSet<br>TreeSet<br>EnumSet|HashMap<br>LinkedHashMap<br>TreeMap<br>IdentityHashMap<br>WeakHashMap<br>EnumMap|
|10|同步的子类|Vector<br>CopyOnWriteArrayList|CopyOnWriteArraySet<br> ConcurrentSkipListSet|Hashtable<br>ConcurrentHashMap<br>ConcurrentSkipListMap|

[http://www.javamadesoeasy.com/2016/02/difference-between-list-set-and-map-in.html](http://www.javamadesoeasy.com/2016/02/difference-between-list-set-and-map-in.html)
