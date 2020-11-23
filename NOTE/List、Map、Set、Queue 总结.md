问：简单说说你理解的 `List` 、`Map`、`Set`、`Queue` 的区别和关系？

答：`List`、`Set`、`Queue` 都继承自 `Collection` 接口，而 `Map` 则不是（继承自 `Object`），
所以**容器类有两个根接口，分别是 `Collection` 和 `Map`，`Collection` 表示单个元素的集合，`Map` 表示键值对的集合。**

## Collection
#### List：可重复，有序
- `ArrayList`：底层数据结构是数组，查询快，增删慢。
***默认初始长度为0，插入第一个数据时，长度为10，满了就扩容，扩容为每次1.5倍。***
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
***默认初始长度为16，默认加载因子为0.75，容量占满75%时扩容，扩容为2倍。***
底层数据结构为 数组 + 链表，为了让数据更均匀的分布在各个数组位置，对key计算hash值，hash计算值相同的数据在同一数组位置，链接形成链表。
```java
// jdk 1.7 中对key计算hash值 并得到数组位置的代码
int hash = hash(key.hashCode());
int i = indexFor(hash, table.length);

static int hash(int h){
	// This function ensures that hashCodes that differ only by
	// constant multiples at each bit position have a bounded
	// number of collisions (approximately 8 at default load factor).
	h ^= (h  > > > 20) ^ (h  > > > 12);
	return h ^ (h  > > > 7) ^ (h  > > > 4);
}

static int indexFor(int h, int length){
	return h & (length-1);
}

```
用`indexFor()`方法，对key值计算`hash & (length-1)`得到的值为位置，但仅用`key.hashCode()`得到的`hash`还是很容易冲突，所以使用`hash()`方法做扰动计算，防止不同hashCode的高位不同但低位相同导致的hash冲突。简单点说，就是为了把高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响。

`hash & (length-1)`其实就是取模运算。
[[JAVA基础#Java中位运算符]]
> 原理如下：
X % 2^n = X & (2^n – 1)
2^n表示2的n次方，也就是说，一个数对2^n取模 == 一个数和(2^n – 1)做按位与运算 。
假设n为3，则2^3 = 8，表示成2进制就是1000。2^3 -1 = 7 ，即0111。
此时X & (2^3 – 1) 就相当于取X的2进制的最后三位数。
从2进制角度来看，X / 8相当于 X >> 3，即把X右移3位，此时得到了X / 8的商，而被移掉的部分(后三位)，则是X % 8，也就是余数。

所以，`return h & (length-1);`只要保证`length`的长度是`2^n`的话，就可以实现取模运算了。而HashMap中的`length`也确实是2的倍数，初始值是16，之后每次扩充为原来的2倍。
[https://www.hollischuang.com/archives/2091](https://www.hollischuang.com/archives/2091)

#### 1.8版本中的HashMap:
在jdk1.8中，引入了红黑树结构做优化，
- 当链表的长度 >= 8时，链表转换为树结构，为什么是8，是因为红黑树的平均查找时间复杂度是log(n)，长度 >= 8开始，红黑树的查找效率优于链表结构，(log(8)=3) < (8/2=4)。
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

[Java 8系列之重新认识HashMap](https://tech.meituan.com/2016/06/24/java-hashmap.html)
[ConcurrentHashMap详解](https://juejin.cn/post/6844904057128091655)