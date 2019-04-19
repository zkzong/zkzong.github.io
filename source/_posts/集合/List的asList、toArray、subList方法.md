---
title: List的asList、toArray、subList方法
date: 2019-04-19
categories: 集合
---

## Arrays.asList
Arrays.asList()把数组转换成集合时，不能使用其修改集合相关的方法，它的 add/remove/clear 方法会抛出 UnsupportedOperationException 异常。 因为asList() 的返回对象是一个 Arrays 内部类，并没有实现集合的修改方法。

## toArray

```
List<String> list = new ArrayList<>(2);
list.add("guan");
list.add("bao");

String[] array = (String[]) list.toArray();
```
直接使用 toArray() 无参方法返回值只能是 Object[]类，若强转其它类型数组将会抛异常。
**解决方案：**使用 `<T> T[] toArray(T[] a);` 有参数这个方法，代码如下：
```
String[] array = new String[list.size()];
array = list.toArray(array);
```

## subList

```
public class SubList {

    @Test
    public void subList() {
        List<String> list = new ArrayList<>();
        list.add("hello");

        // 当原始集合大小没有那么大时，毫无疑问抛异常。
        //java.lang.IndexOutOfBoundsException
        //list.subList(0, 4);

        // 得到一个新的集合，往新集合中增加一条数据。
        List<String> newList = list.subList(0, 1);
        newList.add("zong");

        // 遍历原始集合，竟然 size=2 了，而且往新集合中增加的数据存在于原始集合。
        System.out.println("list size : " + list.size());
        for (String str : list) {
            System.out.println(str);
        }

        // 移除新集合中一条数据，遍历新集合。
        newList.remove("zong");
        for (String str : newList) {
            System.out.println(str);
        }

        System.out.println("-------------------------");
        // 原始集合增加一条数据并遍历。
        list.add("zong");
        for (String str : list) {
            System.out.println(str);
        }

        // 遍历新集合，抛出 ConcurrentModificationException 异常。
        // java.util.ConcurrentModificationException
        for (String str : newList) {
            System.out.println(str);
        }

    }
}
```
返回的新集合是靠原来的集合支持的，修改都会影响到彼此对方。
在 subList 场景中，高度注意对原集合元素个数的修改，会导致子列表的遍历、增加、删除均产生异常。
