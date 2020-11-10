问：简单说说你理解的 `List` 、`Map`、`Set`、`Queue` 的区别和关系？

答：`List`、`Set`、`Queue` 都继承自 `Collection` 接口，而 `Map` 则不是（继承自 `Object`），
所以**容器类有两个根接口，分别是 `Collection` 和 `Map`，`Collection` 表示单个元素的集合，`Map` 表示键值对的集合。**

## Collection
#### List：可重复，有序
- `ArrayList`：底层数据结构是数组，查询快，增删慢
- `LinkedList`：底层数据结构是链表，查询慢，两头增删快
#### Set：不可重复（唯一），无序（TreeSet，LinkedHashSet也是有序的）
- `HashSet`:底层数据结构是hash表（唯一，无序）
- `LinkedHashSet`：底层数据结构是链表和hash表，链表保证有序（唯一，有序）
- `TreeSet`:底层数据结构是红黑树（唯一，有序）
#### Queue
代表了队列，Deque代表了双端队列（即可以作为队列使用，也可以作为栈使用）

## Map
`HashMap`: 无序,基于hash表实现。
有个使用`synchronized`关键字实现线程安全的`HashTable`，效率低。jdk1.5中提供了效率更高的`ConcurrentHashMap`

#### HashMap
底层数据结构为 数组 + 链表，对key值计算`hashcode & (2n-1)`，让数据更均匀的分布在各个数组位置，hash计算值相同的数据在同一数组位置，链接形成链表。

#### 1.8版本中的HashMap:
在jdk1.8中，引入了红黑树结构做优化，
- 当链表的长度 >= 8时，链表转换为树结构，为什么是8，是因为红黑树的平均查找长度是log(n)，长度 >= 8开始，红黑树的查找效率优于链表结构，(log(8)=3) < (8/2=4)。
- 当链表的长度 <= 6时，树结构还原成链表。为什么是6，是因为长度 <=6 时，加上转换为树，生成树的时间，效率并不会比链表结构更高。加上中间隔了个7，避免了频繁增加删除元素时，导致频繁的树与链表的转换。

#### ConcurrentHashMap:
结构图如下：
![[jdk1.7中的ConcurrentHashMap结构图.jpg]]
`ConcurrentHashMap`包含两个静态内部类，`HashEntry`，`Segment`
- `HashEntry`类用来封装映射表的键/值对，类中`key`，`hash` 和 `next` 域都被声明为 `final` 型，`value` 域被声明为 `volatile` 型。
- `Segment`类继承了`ReentrantLock`类，从而使得能用来充当锁的角色。
- `size`计算是先采用不加锁的方式，连续计算元素的个数，最多计算3次：1、如果前后两次计算结果相同，则说明计算出来的元素个数是准确的；2、如果前后两次计算结果都不同，则给每个Segment进行加锁，再计算一次元素的个数。
```java
static class HashEntry<K,V> {
	final K key;
	final int hash;
	volatile V value;
	final HashEntry<K,V> next;

	HashEntry(K key, int hash, HashEntry<K,V> next, V value) {
		this.key = key;
		this.hash = hash;
		this.next = next;
		this.value = value;
	}
}
```

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
	private static final long serialVersionUID = 2249069246763182397L;
	final float loadFactor;
	Segment(float lf) { this.loadFactor = lf; }
}
```

#### 1.8版本中的ConcurrentHashMap:
```java
static class Node<K,V> implements Map.Entry<K,V> {
	final int hash;
	final K key;
	volatile V val;
	volatile Node<K,V> next;

	Node(int hash, K key, V val, Node<K,V> next) {
		this.hash = hash;
		this.key = key;
		this.val = val;
		this.next = next;
	}
}
```
在1.8版本中，放弃了`Segment`臃肿的设计，取而代之的是采用Node + CAS + Synchronized来保证并发安全。使用一个volatile类型的变量`baseCount`记录元素的个数，当插入新数据或则删除数据时，会通过`addCount()`方法更新`baseCount`，通过累加`baseCount`和`CounterCell`数组中的数量，即可得到元素的总个数。

#### TreeMap
有序,是一个有序的key-value集合，基于红黑树的`NavigableMap`实现。TreeMap没有调优选项，因为该树总处于平衡状态。TreeMap中默认是按照升序进行排序的，如何让他降序，通过自定义的比较器`Comparator`来实现