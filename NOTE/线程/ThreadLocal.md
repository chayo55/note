## ThreadLocal是什么？
ThreadLocal提供了线程的局部变量，每个线程都可以通过set()和get()来对这个局部变量进行操作，但不会和其他线程的局部变量进行冲突，实现了**线程的数据隔离**～。
ThreadLocal是一个变量的本地副本，线程对变量的操作不会影响其他线程。
简要言之：**往ThreadLocal中填充的变量属于当前线程，该变量对其他线程而言是隔离的**。

---


## ThreadLocal有哪些应用
Spring中的单例bean，使用`ThreadLoca`l实现了多线程下相同变量的线程安全问题。
[[Transactional#Spring中事务管理|Spring中的事务]]，spring在事务开始时，会使用`ThreadLocal`，将`connection`和当前线程绑定起来，这样在整个事务中都是使用这个`connection`,实现事务的隔离性

---

## ThreadLocalMap
```java
static class ThreadLocalMap {

	static class Entry extends WeakReference<ThreadLocal<?>> {
		/** The value associated with this ThreadLocal. */
		Object value;

		Entry(ThreadLocal<?> k, Object v) {
			super(k);
			value = v;
		}
	}
}
```
`ThreadLocalMap`中用来存储的`Entry<K,V>`，继承了`WeakReference`类，`K`是`ThreadLocal`，`V`是存入的`Value`。调用父类`WeakReference`的构造方法对`K` `ThreadLocal`处理，因此`ThreadLocalMap`中对`ThreadLocal`的引用是**弱引用**.
![[JAVA基础#Java中四大引用]]

### 为什么使用弱引用？
翻看官网文档的说法：                                                                
> To help deal with very large and long-lived usages, the hash table entries use WeakReferences for keys. 
   为了应对大对象和长生命周期对象，哈希表条目的 key 使用 WeakReference。

1. **如果key使用强引用**
引用 `ThreadLocal` 的对象被回收了，但是 `ThreadLocalMap` 还持有 `ThreadLocal` 的强引用。如果没有手动删除 `ThreadLocal` 不会被回收，导致 `Entry` 内存泄漏。

2. **如果key使用弱引用**
引用 `ThreadLocal` 的对象被回收了。由于 `ThreadLocalMap` 持有 `ThreadLocal` 的弱引用，因此即使没有手动删除 `ThreadLocal` 也会被回收。`value` 在下一次 `ThreadLocalMap` 调用` set()、get()` 或 `remove()` 的时候会被清除。

比较两种情况可以发现：由于 `ThreadLocalMap `的生命周期跟 `Thread `一样长，如果都没有手动删除对应 key，都会导致内存泄漏。但是，使用弱引用可以多一层保障：弱引用 `ThreadLocal` 被清理后` key` 为 `null`，对应的  `value`在下一次 `ThreadLocalMap` 调用`set()、get()` 或` remove() `时可能会被清除。


---

## ThreadLocal的内存泄漏问题
### ThreadLocal的内部实现
`ThreadLocal`内部有一个[[内部类与静态内部类(嵌套类)|静态内部类]]`ThreadLocalMap`，并且交由管理的变量，并不存储在`ThreadLocal`，而是存储在`ThreadLocalMap`，它的`key`是`ThreadLocal`实例对象，`value`是交由管理的变量.
#### ThreadLocal源码部分方法：
`set()`方法，获取当前线程的`ThreadLocalMap`，往map保存<K,V>, K是当前`ThreadLocal`的实例，V是传入的`Value`。这里的`map=getMap(t);`是从`Thread`中取的，`Thread`类中维护了一个`ThreadLocalMap`的变量引用.
```java
	public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
	}
```
`get()`方法， 获取当前线程的对应的私有变量，即`set()`方法或`initialValue()`方法设置的值。
```java
	public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```
`remove()`方法，调用`ThreadLocalMap`中的`remove()`方法，
`e.clear()`用于清除`Entry`的`key`，它调用的是`WeakReference`中的方法:`this.referent = null`
`expungeStaleEntry(i)`用于清除Entry对应的value。
```java
	/** ThreadLocal.remove() */
	public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
	 
	 /** ThreadLocalMap.remove() */
	 private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear(); // 清除key
                    expungeStaleEntry(i); // 清除value
                    return;
                }
            }
        }
```
### 内存泄漏发生的根本原因
**由于`ThreadLocalMap`的生命周期跟`Thread`一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。**

---

## ThreadLocal的使用
- ThreadLocal 的场景：
	- 当需要存储线程私有变量的时候；
	- 当需要实现线程安全的变量时；
	- 当需要减少线程资源竞争的时候。

- 如何避免内存泄漏
	- 每次使用完 ThreadLocal，建议调用它的 remove() 方法清除数据
	- 但并不是所有使用 ThreadLocal 的地方都要在最后 remove()，

---

### FastThreadLocal
[FastThreadLocal](https://mp.weixin.qq.com/s/aItosqUu1aMvWqJ2ZMqy5Q)