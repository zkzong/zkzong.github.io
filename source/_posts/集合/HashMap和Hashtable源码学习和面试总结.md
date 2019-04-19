---
title: HashMap和Hashtable源码学习和面试总结
date: 2019-04-19
categories: 集合
---

如果说Java的HashMap是数组+链表，那么JDK 8之后就是数组+链表+红黑树组成了HashMap。

在之前谈过，如果hash算法不好，会使得hash表蜕化为顺序查找，即使负载因子和hash算法优化再多，也无法避免出现链表过长的情景（这个概论虽然很低），于是在JDK1.8中，对HashMap做了优化，引入红黑树。具体原理就是当hash表中每个桶附带的链表长度默认超过8时，链表就转换为红黑树结构，提高HashMap的性能，因为红黑树的增删改是O(logn)，而不是O(n)。

红黑树的具体原理和实现以后再总结。

## 主要看put方法实现

```
public V put(K key, V value) {
	return putVal(hash(key), key, value, false, true);
}
```

封装了一个final方法，里面用到一个常量，具体用处看源码：

```
static final int TREEIFY_THRESHOLD = 8;
```

下面是具体源代码注释：

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0) // 首先判断hash表是否是空的，如果空，则resize扩容
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null) // 通过key计算得到hash表下标，如果下标处为null，就新建链表头结点，在方法最后插入即可
            tab[i] = newNode(hash, key, value, null);
        else { // 如果下标处已经存在节点，则进入到这里
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k)))) // 先看hash表该处的头结点是否和key一样（hashcode和equals比较），一样就更新
                e = p;
            else if (p instanceof TreeNode) // hash表头结点和key不一样，则判断节点是不是红黑树，是红黑树就按照红黑树处理
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else { // 如果不是红黑树，则按照之前的HashMap原理处理
                for (int binCount = 0; ; ++binCount) { // 遍历链表
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st (原jdk注释) 显然当链表长度大于等于7的时候，也就是说大于8的话，就转化为红黑树结构，针对红黑树进行插入（logn复杂度）
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k)))) 
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold) // 如果超过容量，即扩容
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

resize是新的扩容方法，之前谈过，扩容原理是使用新的（2倍旧长度）的数组代替，把旧数组的内容放到新数组，需要重新计算hash和hash表的位置，非常耗时，但是自从 JDK 1.8 对HashMap引入了红黑树，它和之前的扩容方法相比有了改进。

## 扩容方法的改进

```
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && 
                     oldCap >= DEFAULT_INITIAL_CAPACITY) // 如果长度没有超过最大值，则扩容为2倍的关系
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) { // 进行新旧元素的转移过程
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode) 
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order（原注释） 如果不是红黑树的情况这里改进了，没有rehash的过程，如下分别记录链表的头尾
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

因为有这样一个特点：比如hash表的长度是16，那么15对应二进制是：

0000 0000， 0000 0000， 0000 0000， 0000 1111 = 15

扩容之前有两个key，分别是k1和k2：

k1的hash：

0000 0000， 0000 0000， 0000 0000， 0000 1111 = 15

k2的hash：

0000 0000， 0000 0000， 0000 0000， 0001 1111 = 15

hash值和15模得到：

k1：0000 0000， 0000 0000， 0000 0000， 0000 1111 = 15

k2：0000 0000， 0000 0000， 0000 0000， 0000 1111 = 15

扩容之后表长对应为32，则31二进制：

0000 0000， 0000 0000， 0000 0000， 0001 1111 = 31

重新hash之后得到：

k1：0000 0000， 0000 0000， 0000 0000， 0000 1111 = 15

k2：0000 0000， 0000 0000， 0000 0000， 0001 1111 = 31 = 15 + 16

观察发现：如果扩容后新增的位是0，那么rehash索引不变，否则才会改变，并且变为原来的索引+旧hash表的长度，故我们只需看原hash表长新增的bit是1还是0，如果是0，索引不变，如果是1，索引变成原索引+旧表长，根本不用像JDK 7 那样rehash，省去了重新计算hash值的时间，而且新增的bit是0还是1可以认为是随机的，因此resize的过程，还能均匀的把之前的冲突节点分散。 

故JDK 8对HashMap的优化是非常到位的。

---

如下是之前整理的旧hash的实现机制和原理，并和jdk古老的Hashtable做了比较。

**整理jdk 1.8之前的HashMap实现：**

* Java集合概述
* HashMap介绍
* HashMap源码学习
* 关于HashMap的几个经典问题
* Hashtable介绍和源码学习
* HashMap 和 Hashtable比较

先上图

![集合](http://upload-images.jianshu.io/upload_images/292448-c4e1e88ac651c01f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Set和List接口是Collection接口的子接口，分别代表无序集合和有序集合，Queue是Java提供的队列实现。**

![Map](http://upload-images.jianshu.io/upload_images/292448-45a9c007bbd27dda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**Map用于保存具有key-value映射关系的数据。**

Java 中有四种常见的Map实现——HashMap，TreeMap，Hashtable和LinkedHashMap。

* HashMap就是一张hash表，键和值都没有排序。
* TreeMap以红黑树结构为基础，键值可以设置按某种顺序排列。
* LinkedHashMap保存了插入时的顺序。
* Hashtable是同步的(而HashMap是不同步的)。所以如果在线程安全的环境下应该多使用HashMap，而不是Hashtable，因为Hashtable对同步有额外的开销，不过JDK 5之后的版本可以使用conncurrentHashMap代替Hashtable。

本文重点总结HashMap，HashMap是基于哈希表实现的，每一个元素是一个key-value对，其内部通过单链表解决冲突问题，容量不足（超过了阀值）时，同样会自动增长。

HashMap是非线程安全的，只用于单线程环境下，多线程环境下可以采用concurrent并发包下的concurrentHashMap。

HashMap 实现了Serializable接口，因此它支持序列化。

HashMap还实现了Cloneable接口，故能被克隆。


**关于HashMap的用法，这里就不再赘述了，只说原理和一些注意点。**

## HashMap的存储结构

![HashMap的存储结构](http://upload-images.jianshu.io/upload_images/292448-44af1c10c16c5afa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

紫色部分即代表哈希表本身（其实是一个数组），数组的每个元素都是一个单链表的头节点，链表是用来解决hash地址冲突的，如果不同的key映射到了数组的同一位置处，就将其放入单链表中保存。

## HashMap有四个构造方法，方法中有两个很重要的参数：初始容量和加载因子

这两个参数是影响HashMap性能的重要参数，其中容量表示哈希表中槽的数量（即哈希数组的长度），初始容量是创建哈希表时的容量（默认为16），加载因子是哈希表当前key的数量和容量的比值，当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表提前进行 resize 操作（即扩容）。如果加载因子越大，对空间的利用更充分，但是查找效率会降低（链表长度会越来越长）；如果加载因子太小，那么表中的数据将过于稀疏（很多空间还没用，就开始扩容了），严重浪费。

JDK开发者规定的默认加载因子为0.75，因为这是一个比较理想的值。另外，无论指定初始容量为多少，构造方法都会将实际容量设为不小于指定容量的2的幂次方，且最大值不能超过2的30次方。

## 重点分析HashMap中用的最多的两个方法put和get的源码

```
// 获取key对应的value
public V get(Object key) {
	if (key == null)
		return getForNullKey();
	// 获取key的hash值
	int hash = hash(key.hashCode());
	// 在“该hash值对应的链表”上查找“键值等于key”的元素
	for (Entry<K, V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
		Object k;
		// 判断key是否相同
		if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
			return e.value;
	}
	// 没找到则返回null
	return null;
}

// 获取“key为null”的元素的值，HashMap将“key为null”的元素存储在table[0]位置，但不一定是该链表的第一个位置！
private V getForNullKey() {
	for (Entry<K, V> e = table[0]; e != null; e = e.next) {
		if (e.key == null)
			return e.value;
	}
	return null;
}
```

首先，如果key为null，则直接从哈希表的第一个位置table[0]对应的链表上查找。**记住，key为null的键值对永远都放在以table[0]为头结点的链表中，当然不一定是存放在头结点table[0]中。**如果key不为null，则先求的key的hash值，根据hash值找到在table中的索引，在该索引对应的单链表中查找是否有键值对的key与目标key相等，有就返回对应的value，没有则返回null。

```
// 将“key-value”添加到HashMap中
public V put(K key, V value) {
	// 若“key为null”，则将该键值对添加到table[0]中。
	if (key == null)
		return putForNullKey(value);
	// 若“key不为null”，则计算该key的哈希值，然后将其添加到该哈希值对应的链表中。
	int hash = hash(key.hashCode());
	int i = indexFor(hash, table.length);
	for (Entry<K, V> e = table[i]; e != null; e = e.next) {
		Object k;
		// 若“该key”对应的键值对已经存在，则用新的value取代旧的value。然后退出！
		if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
			V oldValue = e.value;
			e.value = value;
			e.recordAccess(this);
			return oldValue;
		}
	}

	// 若“该key”对应的键值对不存在，则将“key-value”添加到table中
	modCount++;
	// 将key-value添加到table[i]处
	addEntry(hash, key, value, i);
	return null;
}
```

如果key为null，则将其添加到table[0]对应的链表中，如果key不为null，则同样先求出key的hash值，根据hash值得出在table中的索引，而后遍历对应的单链表，**如果单链表中存在与目标key相等的键值对，则将新的value覆盖旧的value，且将旧的value返回**，如果找不到与目标key相等的键值对，或者该单链表为空，则将该键值对插入到单链表的头结点位置（每次新插入的节点都是放在头结点的位置），该操作是有addEntry方法实现的，它的源码如下：

```
// 新增Entry。将“key-value”插入指定位置，bucketIndex是位置索引。
void addEntry(int hash, K key, V value, int bucketIndex) {
	// 保存“bucketIndex”位置的值到“e”中
	Entry<K, V> e = table[bucketIndex];
	// 设置“bucketIndex”位置的元素为“新Entry”，
	// 设置“e”为“新Entry的下一个节点”
	table[bucketIndex] = new Entry<K, V>(hash, key, value, e);
	// 若HashMap的实际大小 不小于 “阈值”，则调整HashMap的大小
	if (size++ >= threshold)
		resize(2 * table.length);
}
```

注意这里倒数第三行的构造方法，将key-value键值对赋给table[bucketIndex]，并将其next指向元素e，这便将key-value放到了头结点中，并将之前的头结点接在了它的后面。该方法也说明，每次put键值对的时候，总是将新的该键值对放在table[bucketIndex]处（即头结点处）。两外注意最后两行代码，每次加入键值对时，都要判断当前已用的槽的数目是否大于等于阀值（容量*加载因子），如果大于等于，则进行扩容，将容量扩为原来容量的2倍。

## 重点来分析下求hash值和索引值的方法，这两个方法便是HashMap设计的最为核心的部分，二者结合能保证哈希表中的元素尽可能均匀地散列。

### 由hash值找到对应索引的方法如下

```
static int indexFor(int h, int length) {
	return h & (length-1);
}
```

因为容量初始还是设定都会转化为2的幂次。故可以使用高效的位与运算替代模运算。下面会解释原因。

### 计算hash值的方法如下

```
static int hash(int h) {
	h ^= (h >>> 20) ^ (h >>> 12);
	return h ^ (h >>> 7) ^ (h >>> 4);
}
```

JDK 的 HashMap 使用了一个 hash 方法对hash值使用位的操作，使hash值的计算效率很高。为什么这样做？主要是因为如果直接使用hashcode值，那么这是一个int值（8个16进制数，共32位），int值的范围正负21亿多，但是hash表没有那么长，一般比如初始16，自然散列地址需要对hash表长度取模运算，得到的余数才是地址下标。假设某个key的hashcode是0AAA0000，hash数组长默认16，如果不经过hash函数处理，该键值对会被存放在hash数组中下标为0处，因为0AAA0000 & (16-1) = 0。过了一会儿又存储另外一个键值对，其key的hashcode是0BBB0000，得到数组下标依然是0，这就说明这是个实现得很差的hash算法，因为hashcode的1位全集中在前16位了，导致算出来的数组下标一直是0。于是明明key相差很大的键值对，却存放在了同一个链表里，导致以后查询起来比较慢（蜕化为了顺序查找）。故JDK的设计者使用hash函数的若干次的移位、异或操作，把hashcode的“1位”变得“松散”，非常巧妙。

# 下面是几个常见的面试题

## 说下HashMap的 扩容机制？

前面说了，hashmap的构造器里指明了两个对于理解HashMap比较重要的两个参数 int initialCapacity，float loadFactor，这两个参数会影响HashMap效率，HashMap底层采用的散列数组实现，利用initialCapacity这个参数我们可以设置这个数组的大小，也就是散列桶的数量，但是如果需要Map的数据过多，在不断的add之后，这些桶可能都会被占满，这是有两种策略，一种是不改变Capacity，因为即使桶占满了，我们还是可以利用每个桶附带的链表增加元素。但是这有个缺点，此时HaspMap就退化成为了LinkedList，使get和put方法的时间开销上升，这是就要采用另一种方法：增加Hash桶的数量，这样get和put的时间开销又回退到近于常数复杂度上。Hashmap就是采用的该方法。

### 关于扩容。看HashMap的扩容方法，resize方法，它的源码如下：

```
// 重新调整HashMap的大小，newCapacity是调整后的单位
void resize(int newCapacity) {
	Entry[] oldTable = table;
	int oldCapacity = oldTable.length;
	if (oldCapacity == MAXIMUM_CAPACITY) {
		threshold = Integer.MAX_VALUE;
		return;
	}

	// 新建一个HashMap，将“旧HashMap”的全部元素添加到“新HashMap”中，
	// 然后，将“新HashMap”赋值给“旧HashMap”。
	Entry[] newTable = new Entry[newCapacity];
	transfer(newTable);
	table = newTable;
	threshold = (int) (newCapacity * loadFactor);
}
```

很明显，是从新建了一个HashMap的底层数组，长度为原来的两倍，而后调用transfer方法，将旧HashMap的全部元素添加到新的HashMap中（要重新计算元素在新的数组中的索引位置）。

transfer方法的源码如下：

```
// 将HashMap中的全部元素都添加到newTable中
void transfer(Entry[] newTable) {
	Entry[] src = table;
	int newCapacity = newTable.length;
	for (int j = 0; j < src.length; j++) {
		Entry<K, V> e = src[j];
		if (e != null) {
			src[j] = null;
			do {
				Entry<K, V> next = e.next;
				int i = indexFor(e.hash, newCapacity);
				e.next = newTable[i];
				newTable[i] = e;
				e = next;
			} while (e != null);
		}
	}
}
```

很明显，扩容是一个相当耗时的操作，因为它需要重新计算这些元素在新的数组中的位置并进行复制处理。因此，我们在用HashMap时，最好能提前预估下HashMap中元素的个数，这样有助于提高HashMap的性能。

## HashMap什么时候需要增加容量呢？

因为效率问题，JDK采用预处理法，这时前面说的loadFactor就派上了用场，当size > initialCapacity * loadFactor，HashMap内部resize方法就被调用，使得重新扩充hash桶的数量，在目前的实现中，是增加一倍，这样就保证当你真正想put新的元素时效率不会明显下降。所以一般情况下HashMap并不存在键值放满的情况。当然并不排除极端情况，比如设置的JVM内存用完了，或者这个HashMap的Capacity已经达到了MAXIMUM_CAPACITY（目前的实现是2^30）。

## initialCapacity和loadFactor参数设什么样的值好呢？

initialCapacity的默认值是16，有些人可能会想如果内存足够，是不是可以将initialCapacity设大一些，即使用不了这么大，就可避免扩容导致的效率的下降，反正无论initialCapacity大小，我们使用的get和put方法都是常数复杂度的。这么说没什么不对，但是可能会忽略一点，实际的程序可能不仅仅使用get和put方法，也有可能使用迭代器，如initialCapacity容量较大，那么会使迭代器效率降低。所以理想的情况还是在使用HashMap前估计一下数据量。

加载因子默认值是0.75，是JDK权衡时间和空间效率之后得到的一个相对优良的数值。如果这个值过大，虽然空间利用率是高了，但是对于HashMap中的一些方法的效率就下降了，包括get和put方法，会导致每个hash桶所附加的链表增长，影响存取效率。如果比较小，除了导致空间利用率较低外没有什么坏处，只要有的是内存，毕竟现在大多数人把时间看的比空间重要。但是实际中还是很少有人会将这个值设置的低于0.5。

## HashMap的key和value都能为null么？如果k能为null，那么它是怎么样查找值的？

如果key为null，则直接从哈希表的第一个位置table[0]对应的链表上查找。记住，key为null的键值对永远都放在以table[0]为头结点的链表中。

## HashMap中put值的时候如果发生了冲突，是怎么处理的？

JDK使用了链地址法，hash表的每个元素又分别链接着一个单链表，元素为头结点，如果不同的key映射到了相同的下标，那么就使用头插法，插入到该元素对应的链表。

## HashMap的key是如何散列到hash表的？相比较Hashtable有什么改进？

我们一般对哈希表的散列很自然地会想到用hash值对length取模（即除留余数法），Hashtable就是这样实现的，这种方法基本能保证元素在哈希表中散列的比较均匀，但取模会用到除法运算，效率很低，且Hashtable直接使用了hashcode值，没有重新计算。

HashMap中则通过 h&(length-1) 的方法来代替取模，其中h是key的hash值，同样实现了均匀的散列，但效率要高很多，这也是HashMap对Hashtable的一个改进。

接下来，我们分析下为什么哈希表的容量一定要是2的整数次幂。

首先，length为2的整数次幂的话，h&(length-1) 在数学上就相当于对length取模，这样便保证了散列的均匀，同时也提升了效率；

其次，length为2的整数次幂的话，则一定为偶数，那么 length-1 一定为奇数，奇数的二进制的最后一位是1，这样便保证了 h&(length-1) 的最后一位可能为0，也可能为1（这取决于h的值），即与后的结果可能为偶数，也可能为奇数，这样便可以保证散列的均匀，而如果length为奇数的话，很明显 length-1 为偶数，它的最后一位是0，这样 h&(length-1) 的最后一位肯定为0，即只能为偶数，这样导致了任何hash值都只会被散列到数组的偶数下标位置上，浪费了一半的空间，因此length取2的整数次幂，是为了使不同hash值发生碰撞的概率较小，这样就能使元素在哈希表中均匀地散列。

# 作为对比，再讨论一下Hashtable

Hashtable同样是基于哈希表实现的，其实类似HashMap，只不过有些区别，Hashtable同样每个元素是一个key-value对，其内部也是通过单链表解决冲突问题，容量不足（超过了阀值）时，同样会自动增长。

Hashtable比较古老， 是JDK1.0就引入的类，而HashMap 是 1.2 引进的 Map 的一个实现。

Hashtable是线程安全的，能用于多线程环境中。Hashtable同样也实现了Serializable接口，支持序列化，也实现了Cloneable接口，能被克隆。

![Hashtable](https://upload-images.jianshu.io/upload_images/292448-fde29168d2ef38df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Hashtable继承于Dictionary类，实现了Map接口。Dictionary是声明了操作"键值对"函数接口的抽象类。 有一点注意，Hashtable除了线程安全之外（其实是直接在方法上增加了synchronized关键字，比较古老，落后，低效的同步方式），还有就是它的key、value都不为null。另外Hashtable 也有 **初始容量** 和 **加载因子**。

```
public Hashtable() {
	this(11, 0.75f);
}
```

**默认加载因子也是 0.75，Hashtable在不指定容量的情况下的默认容量为11，而HashMap为16，Hashtable不要求底层数组的容量一定要为2的整数次幂，而HashMap则要求一定为2的整数次幂。因为Hashtable是直接使用除留余数法定位地址。且Hashtable计算hash值，直接用key的hashCode()。**

还要注意：前面说了Hashtable中key和value都不允许为null，而HashMap中key和value都允许为null（key只能有一个为null，而value则可以有多个为null）。但如在Hashtable中有类似put(null,null)的操作，编译同样可以通过，因为key和value都是Object类型，但运行时会抛出NullPointerException异常，这是JDK的规范规定的。

最后针对扩容：**Hashtable扩容时，将容量变为原来的2倍加1，而HashMap扩容时，将容量变为原来的2倍。**

# 下面是几个常见的笔试，面试题

## Hashtable和HashMap的区别有哪些？

HashMap和Hashtable都实现了Map接口，但决定用哪一个之前先要弄清楚它们之间的分别。主要的区别有：线程安全性，同步(synchronization)，以及速度。

理解HashMap是Hashtable的轻量级实现（非线程安全的实现，Hashtable是非轻量级，线程安全的），都实现Map接口，主要区别在于：

1. 由于HashMap非线程安全，在只有一个线程访问的情况下，效率要高于Hashtable。
2. HashMap允许将null作为一个entry的key或者value，而Hashtable不允许。
3. HashMap把Hashtable的contains方法去掉了，改成containsValue和containsKey。因为contains方法容易让人引起误解。
4. Hashtable继承自陈旧的Dictionary类，而HashMap是Java1.2引进的Map 的一个实现。
5. Hashtable和HashMap扩容的方法不一样，Hashtable中hash数组默认大小11，扩容方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数，增加为原来的2倍，没有加1。
6. 两者通过hash值散列到hash表的算法不一样，HashTbale是古老的除留余数法，直接使用hashcode，而后者是强制容量为2的幂，重新根据hashcode计算hash值，在使用hash  位与  （hash表长度 – 1），也等价取模，但更加高效，取得的位置更加分散，偶数，奇数保证了都会分散到。前者就不能保证。
7. 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。

* fail-fast和iterator迭代器相关。如果某个集合对象创建了Iterator或者ListIterator，然后其它的线程试图“结构上”更改集合对象，将会抛出ConcurrentModificationException异常。但其它线程可以通过set()方法更改集合对象是允许的，因为这并没有从“结构上”更改集合。但是假如已经从结构上进行了更改，再调用set()方法，将会抛出IllegalArgumentException异常。 
* 结构上的更改指的是删除或者插入一个元素，这样会影响到map的结构。
* 该条说白了就是在使用迭代器的过程中有其他线程在结构上修改了map，那么将抛出ConcurrentModificationException，这就是所谓fail-fast策略。

## 为什么HashMap是线程不安全的，实际会如何体现？

第一，如果多个线程同时使用put方法添加元素

假设正好存在两个put的key发生了碰撞(hash值一样)，那么根据HashMap的实现，这两个key会添加到数组的同一个位置，这样最终就会发生其中一个线程的put的数据被覆盖。

第二，如果多个线程同时检测到元素个数超过数组大小*loadFactor

这样会发生多个线程同时对hash数组进行扩容，都在重新计算元素位置以及复制数据，但是最终只有一个线程扩容后的数组会赋给table，也就是说其他线程的都会丢失，并且各自线程put的数据也丢失。**且会引起死循环的错误。**

具体细节上的原因，可以参考：[不正当使用HashMap导致cpu 100%的问题追究](http://ifeve.com/hashmap-infinite-loop/)

## 能否让HashMap实现线程安全，如何做？

1、直接使用Hashtable，但是当一个线程访问Hashtable的同步方法时，其他线程如果也要访问同步方法，会被阻塞住。举个例子，当一个线程使用put方法时，另一个线程不但不可以使用put方法，连get方法都不可以，效率很低，现在基本不会选择它了。

2、HashMap可以通过下面的语句进行同步：

<pre style="margin: 0px; padding: 0px; white-space: pre-wrap; overflow-wrap: break-word; font-family: &quot;Courier New&quot; !important; font-size: 12px !important;">Collections.synchronizeMap(hashMap);</pre>

3、直接使用JDK 5 之后的 ConcurrentHashMap，如果使用Java 5或以上的话，请使用ConcurrentHashMap。

## Collections.synchronizeMap(hashMap);又是如何保证了HashMap线程安全？

直接分析源码吧

![image](http://upload-images.jianshu.io/upload_images/292448-4595b9ea762e2cbe.gif?imageMogr2/auto-orient/strip)

 View Code

从源码中看出 synchronizedMap()方法返回一个SynchronizedMap类的对象，而在SynchronizedMap类中使用了synchronized来保证对Map的操作是线程安全的，故效率其实也不高。

## 为什么Hashtable的默认大小和HashMap不一样？

前面分析了，Hashtable 的扩容方法是乘2再+1，不是简单的乘2，故hashtable保证了容量永远是奇数，结合之前分析HashMap的重算hash值的逻辑，就明白了，因为在数据分布在等差数据集合(如偶数)上时，如果公差与桶容量有公约数 n，则至少有(n-1)/n 数量的桶是利用不到的，故之前的HashMap会在取模（使用位与运算代替）哈希前先做一次哈希运算，调整hash值。这里Hashtable比较古老，直接使用了除留余数法，那么就需要设置容量起码不是偶数（除（近似）质数求余的分散效果好）。而JDK开发者选了11。

## JDK 8对HashMap有了什么改进？说说你对红黑树的理解？

参考更新的jdk 8对HashMap的的改进部分整理，并且还能引申出高级数据结构——红黑树，这又能引出很多问题……学无止境啊！

临时小结：感觉针对Java的HashMap和Hashtable面试，或者理解，到这里就可以了，具体就是多写代码实践。
