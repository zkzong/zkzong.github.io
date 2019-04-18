| \ |HashSet|LinkedHashSet|TreeSet|
|:--:|--|--|--|
|内部工作机制|HashSet内部使用HashMap存储元素|LinkedHashSet内部使用LinkedHashMap存储元素|TreeSet内部使用TreeMap存储元素|
|元素顺序|HashSet不维护元素的顺序|LinkedHashSet维护元素的插入顺序，元素按插入顺序排序|TreeSet根据提供的Comparator排序。如果没有Comparator，元素按自然升序排序|
|性能|HashSet性能比LinkedHashSet和TreeSet都好|LinkedHashSet的性能介于HashSet和TreeSet之间。不过和HashSet接近，只是稍微慢一点。因为它使用LinkedList来维护元素的插入顺序|TreeSet在三者中性能最差，因为在插入和删除操作之后会对元素排序|
|插入、删除、查找操作|时间复杂度：O(1)|时间复杂度：O(1)|时间复杂度：O(log(n))|
|如何比较元素|HashSet使用equals()和hashCode()比较元素是否重复|LinkedHashSet也使用equals()和hashCode()比较元素是否重复|TreeSet使用compare()或compareTo()方法比较元素是否重复，而不是使用equals()和hashCode()|
|null元素|HashSet允许有一个null元素|LinkedHashSet允许有一个null元素|TreeSet不允许有null元素|
|内存占用|HashSet需要最少的内存，因为它使用HashMap存储元素|LinkedHashSet需要的内存比HashSet多，因为它需要维护LinkedList而且使用HashMap存储元素|TreeSet需要的内存也比HashSet多，因为它需要维护Comparator对元素排序并使用TreeMap存储元素|
|何时使用|不需要元素的顺序时使用HashSet|如果需要维护插入顺序则使用LinkedHashSet|如果需要根据某些Comparator排序则使用TreeSet|

HashSet、LinkedHashSet和TreeSet的**相同点**：

+ 不允许重复元素
+ 非同步（线程不安全）
+ 继承Cloneable和Serializable类

[http://javaconceptoftheday.com/hashset-vs-linkedhashset-vs-treeset-in-java/](http://javaconceptoftheday.com/hashset-vs-linkedhashset-vs-treeset-in-java/)
