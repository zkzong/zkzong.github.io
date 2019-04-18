foreach遍历list集合删除某些元素一定会报错吗？
先上一段代码：
```
List list = new ArrayList();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
list.add("5");
for (String item : list) {
    if (item.equals("3")) {
        System.out.println(item);
        list.remove(item);
    }
}
System.out.println(list.size());
```
控制台报错：java.util.ConcurrentModificationException。
这是怎么回事，然后去看了看这个异常，才发现自己果然还是太年轻啊。
我们都知道增加for循环即foreach循环其实就是根据list对象创建一个iterator迭代对象，用这个迭代对象来遍历list，相当于list对象中元素的遍历托管给了iterator，如果要对list进行增删操作，都必须经过iterator。

每次foreach循环时都有以下两个操作：
1. iterator.hasNext(); //判读是否有下个元素
2. item = iterator.next(); //下个元素是什么，并把它赋给item。

首先，我们来看看这个异常信息是什么。
```
public boolean hasNext() {
	return cursor != size;
}

@SuppressWarnings("unchecked")
public E next() {
	checkForComodification(); // 此处报错
	int i = cursor;
	if (i >= size)
		throw new NoSuchElementException();
	Object[] elementData = ArrayList.this.elementData;
	if (i >= elementData.length)
		throw new ConcurrentModificationException();
	cursor = i + 1;
	return (E) elementData[lastRet = i];
}

final void checkForComodification() {
	if (modCount != expectedModCount)
		throw new ConcurrentModificationException();
}
```
可以看到是进入checkForComodification()方法的时候报错了，也就是说modCount != expectedModCount。具体的原因是：以foreach方式遍历元素的时候，会生成iterator，然后使用iterator遍历。在生成iterator的时候，会保存一个expectedModCount参数，这个是生成iterator的时候List中修改元素的次数。如果你在遍历过程中删除元素，List中modCount就会变化，如果这个modCount和exceptedModCount不一致，就会抛出异常，这个是为了安全考虑。

看看list的remove源码：
```
public boolean remove(Object o) {
	if (o == null) {
		for (int index = 0; index < size; index++)
			if (elementData[index] == null) {
				fastRemove(index);
				return true;
			}
	} else {
		for (int index = 0; index < size; index++)
			if (o.equals(elementData[index])) {
				fastRemove(index);
				return true;
			}
	}
	return false;
}
```
看，并没有对expectedModCount进行任何修改，导致expectedModCount和modCount不一致，抛出异常。

但是，遍历list删除元素使用Iterator则不会报错，如下：
```
Iterator it = list.iterator();
while (it.hasNext()) {
	if (it.next().equals("3")) {
		it.remove();
	}
}
```
看看Iterator的remove()方法的源码，是对expectedModCount重新做了赋值处理的，如下：
```
public void remove() {
	if (lastRet < 0)
		throw new IllegalStateException();
	checkForComodification();
	
	try {
		ArrayList.this.remove(lastRet);
		cursor = lastRet;
		lastRet = -1;
		expectedModCount = modCount; // 处理expectedModCount
	} catch (IndexOutOfBoundsException ex) {
		throw new ConcurrentModificationException();
	}
}
```
这样的话保持expectedModCount = modCount相等，就不会报出错了。

**是不是foreach所有的list删除操作都会报出这个错呢？**

其实不一定。如果删除的元素是倒数第二个数的话，其实是不会报错的。为什么呢，来一起看看。
之前说了foreach循环会走两个方法hasNext() 和next()。如果不想报错的话，只要不进next()方法就好啦，看看hasNext()的方法。
```
public boolean hasNext() {
	return cursor != size;
}
```
那么就要求hasNext()的方法返回false了，即cursor == size。其中cursor是Itr类（Iterator子类）中的一个字段，用来保存当前iterator的位置信息，从0开始。cursor本身就是游标的意思，在数据库的操作中用的比较多。只要curosr不等于size就认为存在元素。由于Itr是ArrayList的内部类，因此直接调用了ArrayList的size字段，所以这个字段的值是动态变化的，既然是动态变化的可能就会有问题出现了。
我们以上面的代码为例，当到倒数第二个数据也就是“4”的时候，cursor是4，然后调用删除操作，此时size由5变成了4，当再调用hasNext判断的时候，cursor==size，就会调用后面的操作直接退出循环了。我们可以在上面的代码添加一行代码查看效果：
```
for (String item : list) {
	System.out.println(item);
	if (item.equals("4")) {
		list.remove(item);
	}
}
```
输出是：
```
1
2
3
4
```
这样的话就可以看到执行到hasNext()方法就退出了，也就不会走后面的异常了。
由此可以得出，用foreach删除list元素的时候只有倒数第二个元素删除不会报错，其他都会报错，所以**删除list元素时一定要用Iterator**。

参考文献：
[https://blog.csdn.net/bimuyulaila/article/details/52088124](https://blog.csdn.net/bimuyulaila/article/details/52088124)
