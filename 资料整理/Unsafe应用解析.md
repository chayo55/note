# [Java 魔法类：Unsafe 应用解析](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)

## 前言

Unsafe 是位于 sun.misc 包下的一个类，主要提供一些用于执行低级别、不安全操作的方法，如直接访问系统内存资源、自主管理内存资源等，这些方法在提升 Java 运行效率、增强 Java 语言底层资源操作能力方面起到了很大的作用。但由于 Unsafe 类使 Java 语言拥有了类似 C 语言指针一样操作内存空间的能力，这无疑也增加了程序发生相关指针问题的风险。在程序中过度、不正确使用 Unsafe 类会使得程序出错的概率变大，使得 Java 这种安全的语言变得不再“安全”，因此对 Unsafe 的使用一定要慎重。

注：本文对 sun.misc.Unsafe 公共 API 功能及相关应用场景进行介绍。

## 基本介绍

如下 Unsafe 源码所示，Unsafe 类为一单例实现，提供静态方法 getUnsafe 获取 Unsafe 实例，当且仅当调用 getUnsafe 方法的类为引导类加载器所加载时才合法，否则抛出 
SecurityException 异常。

```Java
	public final class Unsafe {
	  // 单例对象
	  private static final Unsafe theUnsafe;

	  private Unsafe() {
	  }
	  @CallerSensitive
	  public static Unsafe getUnsafe() {
		Class var0 = Reflection.getCallerClass();
		// 仅在引导类加载器`BootstrapClassLoader`加载时才合法
		if(!VM.isSystemDomainLoader(var0.getClassLoader())) {
		  throw new SecurityException("Unsafe");
		} else {
		  return theUnsafe;
		}
	  }
	}
```
	
那如若想使用这个类，该如何获取其实例？有如下两个可行方案。

其一，从`getUnsafe`方法的使用限制条件出发，通过 Java 命令行命令`-Xbootclasspath/a`把调用 Unsafe 相关方法的类 A 所在 jar 包路径追加到默认的 bootstrap 路径中，使得 A 被引导类加载器加载，从而通过`Unsafe.getUnsafe`方法安全的获取 Unsafe 实例。

    java -Xbootclasspath/a: ${path}   // 其中path为调用Unsafe相关方法的类所在jar包路径

其二，通过反射获取单例对象 theUnsafe。

```Java
private static Unsafe reflectGetUnsafe() {
	try {
	  Field field = Unsafe.class.getDeclaredField("theUnsafe");
	  field.setAccessible(true);
	  return (Unsafe) field.get(null);
	} catch (Exception e) {
	  log.error(e.getMessage(), e);
	  return null;
	}
}
```

## 功能介绍

![[Unsafe.png]]

如上图所示，Unsafe 提供的 API 大致可分为内存操作、CAS、Class 相关、对象操作、线程调度、系统信息获取、内存屏障、数组操作等几类，下面将对其相关方法和应用场景进行详细介绍。

### 内存操作

这部分主要包含堆外内存的分配、拷贝、释放、给定地址值操作等方法。

```Java
	//分配内存, 相当于C++的malloc函数
	public native long allocateMemory(long bytes);
	//扩充内存
	public native long reallocateMemory(long address, long bytes);
	//释放内存
	public native void freeMemory(long address);
	//在给定的内存块中设置值
	public native void setMemory(Object o, long offset, long bytes, byte value);
	//内存拷贝
	public native void copyMemory(Object srcBase, long srcOffset, Object destBase, long destOffset, long bytes);
	//获取给定地址值，忽略修饰限定符的访问限制。与此类似操作还有: getInt，getDouble，getLong，getChar等
	public native Object getObject(Object o, long offset);
	//为给定地址设置值，忽略修饰限定符的访问限制，与此类似操作还有: putInt,putDouble，putLong，putChar等
	public native void putObject(Object o, long offset, Object x);
	//获取给定地址的byte类型的值（当且仅当该内存地址为allocateMemory分配时，此方法结果为确定的）
	public native byte getByte(long address);
	//为给定地址设置byte类型的值（当且仅当该内存地址为allocateMemory分配时，此方法结果才是确定的）
	public native void putByte(long address, byte x);
```

通常，我们在 Java 中创建的对象都处于堆内内存（heap）中，堆内内存是由 JVM 所管控的 Java 进程内存，并且它们遵循 JVM 的内存管理机制，[[JVM]] 会采用垃圾回收机制统一管理堆内存。与之相对的是堆外内存，存在于 JVM 管控之外的内存区域，Java 中对堆外内存的操作，依赖于 Unsafe 提供的操作堆外内存的 native 方法。

#### 使用堆外内存的原因

- 对垃圾回收停顿的改善。由于堆外内存是直接受操作系统管理而不是 JVM，所以当我们使用堆外内存时，即可保持较小的堆内内存规模。从而在 GC 时减少回收停顿对于应用的影响。
- 提升程序 I/O 操作的性能。通常在 I/O 通信过程中，会存在堆内内存到堆外内存的数据拷贝操作，对于需要频繁进行内存间数据拷贝且生命周期较短的暂存数据，都建议存储到堆外内存。

#### 典型应用

DirectByteBuffer 是 Java 用于实现堆外内存的一个重要类，通常用在通信过程中做缓冲池，如在 Netty、MINA 等 NIO 框架中应用广泛。DirectByteBuffer 对于堆外内存的创建、使用、销毁等逻辑均由 Unsafe 提供的堆外内存 API 来实现。

下图为 DirectByteBuffer 构造函数，创建 DirectByteBuffer 的时候，通过 Unsafe.allocateMemory 分配内存、Unsafe.setMemory 进行内存初始化，而后构建 Cleaner 对象用于跟踪 DirectByteBuffer 对象的垃圾回收，以实现当 DirectByteBuffer 被垃圾回收时，分配的堆外内存一起被释放。

![[DirectByteBuffer.png]]

那么如何通过构建垃圾回收追踪对象 Cleaner 实现堆外内存释放呢？

Cleaner 继承自 Java 四大引用类型之一的虚引用 PhantomReference（众所周知，无法通过虚引用获取与之关联的对象实例，且当对象仅被虚引用引用时，在任何发生 GC 的时候，其均可被回收），通常 PhantomReference 与引用队列 ReferenceQueue 结合使用，可以实现虚引用关联对象被垃圾回收时能够进行系统通知、资源清理等功能。如下图所示，当某个被 Cleaner 引用的对象将被回收时，JVM 垃圾收集器会将此对象的引用放入到对象引用中的 pending 链表中，等待 Reference-Handler 进行相关处理。其中，Reference-Handler 为一个拥有最高优先级的守护线程，会循环不断的处理 pending 链表中的对象引用，执行 Cleaner 的 clean 方法进行相关清理工作。

![[Cleaner.png]]

所以当 DirectByteBuffer 仅被 Cleaner 引用（即为虚引用）时，其可以在任意 GC 时段被回收。当 DirectByteBuffer 实例对象被回收时，在 Reference-Handler 线程操作中，会调用 Cleaner 的 clean 方法根据创建 Cleaner 时传入的 Deallocator 来进行堆外内存的释放。

![[Cleaner.class.png]]

### CAS 相关

如下源代码释义所示，这部分主要为 CAS 相关操作的方法。

```Java
/**
	*  CAS
  * @param o         包含要修改field的对象
  * @param offset    对象中某field的偏移量
  * @param expected  期望值
  * @param update    更新值
  * @return          true | false
  */
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);

public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);

public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);
```

什么是 CAS? 即比较并替换，实现并发算法时常用到的一种技术。CAS 操作包含三个操作数——内存位置、预期原值及新值。执行 CAS 操作的时候，将内存位置的值与预期原值比较，如果相匹配，那么处理器会自动将该位置值更新为新值，否则，处理器不做任何操作。我们都知道，CAS 是一条 CPU 的原子指令（cmpxchg 指令），不会造成所谓的数据不一致问题，Unsafe 提供的 CAS 方法（如 compareAndSwapXXX）底层实现即为 CPU 指令 cmpxchg。

#### 典型应用

CAS 在 java.util.concurrent.atomic 相关类、Java AQS、CurrentHashMap 等实现上有非常广泛的应用。如下图所示，AtomicInteger 的实现中，静态字段 valueOffset 即为字段 value 的内存偏移地址，valueOffset 的值在 AtomicInteger 初始化时，在静态代码块中通过 Unsafe 的 objectFieldOffset 方法获取。在 AtomicInteger 中提供的线程安全方法中，通过字段 valueOffset 的值可以定位到 AtomicInteger 对象中 value 的内存地址，从而可以根据 CAS 实现对 value 字段的原子操作。

![[AtomicInteger.png]]

下图为某个 AtomicInteger 对象自增操作前后的内存示意图，对象的基地址 baseAddress=“0x110000”，通过 baseAddress+valueOffset 得到 value 的内存地址 valueAddress=“0x11000c”；然后通过 CAS 进行原子性的更新操作，成功则返回，否则继续重试，直到更新成功为止。

![[AtomicInteger CAS.png]]

### 线程调度

这部分，包括线程挂起、恢复、锁机制等方法。

```Java
//取消阻塞线程
public native void unpark(Object thread);
//阻塞线程
public native void park(boolean isAbsolute, long time);
//获得对象锁（可重入锁）
@Deprecated
public native void monitorEnter(Object o);
//释放对象锁
@Deprecated
public native void monitorExit(Object o);
//尝试获取对象锁
@Deprecated
public native boolean tryMonitorEnter(Object o);
```

如上源码说明中，方法 park、unpark 即可实现线程的挂起与恢复，将一个线程进行挂起是通过 park 方法实现的，调用 park 方法后，线程将一直阻塞直到超时或者中断等条件出现；unpark 可以终止一个挂起的线程，使其恢复正常。

#### 典型应用

Java 锁和同步器框架的核心类 AbstractQueuedSynchronizer，就是通过调用`LockSupport.park()`和`LockSupport.unpark()`实现线程的阻塞和唤醒的，而 LockSupport 的 park、unpark 方法实际是调用 Unsafe 的 park、unpark 方式来实现。

### Class 相关

此部分主要提供 Class 和它的静态字段的操作相关方法，包含静态字段内存定位、定义类、定义匿名类、检验&确保初始化等。

```Java
//获取给定静态字段的内存地址偏移量，这个值对于给定的字段是唯一且固定不变的
public native long staticFieldOffset(Field f);
//获取一个静态类中给定字段的对象指针
public native Object staticFieldBase(Field f);
//判断是否需要初始化一个类，通常在获取一个类的静态属性的时候（因为一个类如果没初始化，它的静态属性也不会初始化）使用。 当且仅当ensureClassInitialized方法不生效时返回false。
public native boolean shouldBeInitialized(Class<?> c);
//检测给定的类是否已经初始化。通常在获取一个类的静态属性的时候（因为一个类如果没初始化，它的静态属性也不会初始化）使用。
public native void ensureClassInitialized(Class<?> c);
//定义一个类，此方法会跳过JVM的所有安全检查，默认情况下，ClassLoader（类加载器）和ProtectionDomain（保护域）实例来源于调用者
public native Class<?> defineClass(String name, byte[] b, int off, int len, ClassLoader loader, ProtectionDomain protectionDomain);
//定义一个匿名类
public native Class<?> defineAnonymousClass(Class<?> hostClass, byte[] data, Object[] cpPatches);
```

#### 典型应用

从 Java 8 开始，JDK 使用 invokedynamic 及 VM Anonymous Class 结合来实现 Java 语言层面上的 Lambda 表达式。

- **invokedynamic**： invokedynamic 是 Java 7 为了实现在 JVM 上运行动态语言而引入的一条新的虚拟机指令，它可以实现在运行期动态解析出调用点限定符所引用的方法，然后再执行该方法，invokedynamic 指令的分派逻辑是由用户设定的引导方法决定。
- **VM Anonymous Class**：可以看做是一种模板机制，针对于程序动态生成很多结构相同、仅若干常量不同的类时，可以先创建包含常量占位符的模板类，而后通过 Unsafe.defineAnonymousClass 方法定义具体类时填充模板的占位符生成具体的匿名类。生成的匿名类不显式挂在任何 ClassLoader 下面，只要当该类没有存在的实例对象、且没有强引用来引用该类的 Class 对象时，该类就会被 GC 回收。故而 VM Anonymous Class 相比于 Java 语言层面的匿名内部类无需通过 ClassClassLoader 进行类加载且更易回收。

在 Lambda 表达式实现中，通过 invokedynamic 指令调用引导方法生成调用点，在此过程中，会通过 ASM 动态生成字节码，而后利用 Unsafe 的 defineAnonymousClass 方法定义实现相应的函数式接口的匿名类，然后再实例化此匿名类，并返回与此匿名类中函数式方法的方法句柄关联的调用点；而后可以通过此调用点实现调用相应 Lambda 表达式定义逻辑的功能。下面以如下图所示的 Test 类来举例说明。

![[Unsafe Lambda.png]]

Test 类编译后的 class 文件反编译后的结果如下图一所示（删除了对本文说明无意义的部分），我们可以从中看到 main 方法的指令实现、invokedynamic 指令调用的引导方法 BootstrapMethods、及静态方法`lambda$main$0`（实现了 Lambda 表达式中字符串打印逻辑）等。在引导方法执行过程中，会通过 Unsafe.defineAnonymousClass 生成如下图二所示的实现 Consumer 接口的匿名类。其中，accept 方法通过调用 Test 类中的静态方法`lambda$main$0`来实现 Lambda 表达式中定义的逻辑。而后执行语句`consumer.accept（"lambda"）`其实就是调用下图二所示的匿名类的 accept 方法。

![[Unsafe Lambda 反编译.png]]

### 对象操作

此部分主要包含对象成员属性相关操作及非常规的对象实例化方式等相关方法。

```Java
//返回对象成员属性在内存地址相对于此对象的内存地址的偏移量
public native long objectFieldOffset(Field f);
//获得给定对象的指定地址偏移量的值，与此类似操作还有：getInt，getDouble，getLong，getChar等
public native Object getObject(Object o, long offset);
//给定对象的指定地址偏移量设值，与此类似操作还有：putInt，putDouble，putLong，putChar等
public native void putObject(Object o, long offset, Object x);
//从对象的指定偏移量处获取变量的引用，使用volatile的加载语义
public native Object getObjectVolatile(Object o, long offset);
//存储变量的引用到对象的指定的偏移量处，使用volatile的存储语义
public native void putObjectVolatile(Object o, long offset, Object x);
//有序、延迟版本的putObjectVolatile方法，不保证值的改变被其他线程立即看到。只有在field被volatile修饰符修饰时有效
public native void putOrderedObject(Object o, long offset, Object x);
//绕过构造方法、初始化代码来创建对象
public native Object allocateInstance(Class<?> cls) throws InstantiationException;
```

#### 典型应用

- **常规对象实例化方式**：我们通常所用到的创建对象的方式，从本质上来讲，都是通过 new 机制来实现对象的创建。但是，new 机制有个特点就是当类只提供有参的构造函数且无显示声明无参构造函数时，则必须使用有参构造函数进行对象构造，而使用有参构造函数时，必须传递相应个数的参数才能完成对象实例化。
- **非常规的实例化方式**：而 Unsafe 中提供 allocateInstance 方法，仅通过 Class 对象就可以创建此类的实例对象，而且不需要调用其构造函数、初始化代码、JVM 安全检查等。它抑制修饰符检测，也就是即使构造器是 private 修饰的也能通过此方法实例化，只需提类对象即可创建相应的对象。由于这种特性，allocateInstance 在 java.lang.invoke、Objenesis（提供绕过类构造器的对象生成方式）、Gson（反序列化时用到）中都有相应的应用。

如下图所示，在 Gson 反序列化时，如果类有默认构造函数，则通过反射调用默认构造函数创建实例，否则通过 UnsafeAllocator 来实现对象实例的构造，UnsafeAllocator 通过调用 Unsafe 的 allocateInstance 实现对象的实例化，保证在目标类无默认构造函数时，反序列化不够影响。

![[Unsafe非常规的实例化方式.png]]

### 数组相关

这部分主要介绍与数据操作相关的 arrayBaseOffset 与 arrayIndexScale 这两个方法，两者配合起来使用，即可定位数组中每个元素在内存中的位置。

```Java
//返回数组中第一个元素的偏移地址
public native int arrayBaseOffset(Class<?> arrayClass);
//返回数组中一个元素占用的大小
public native int arrayIndexScale(Class<?> arrayClass);
```

#### 典型应用

这两个与数据操作相关的方法，在 java.util.concurrent.atomic 包下的 AtomicIntegerArray（可以实现对 Integer 数组中每个元素的原子性操作）中有典型的应用，如下图 AtomicIntegerArray 源码所示，通过 Unsafe 的 arrayBaseOffset、arrayIndexScale 分别获取数组首元素的偏移地址 base 及单个元素大小因子 scale。后续相关原子性操作，均依赖于这两个值进行数组中元素的定位，如下图二所示的 getAndAdd 方法即通过 checkedByteOffset 方法获取某数组元素的偏移地址，而后通过 CAS 实现原子性操作。

![[AtomicIntegerArray中unsafe的应用.png]]

### 内存屏障

在 Java 8 中引入，用于定义内存屏障（也称内存栅栏，内存栅障，屏障指令等，是一类同步屏障指令，是 CPU 或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作），避免代码重排序。

```Java
//内存屏障，禁止load操作重排序。屏障前的load操作不能被重排序到屏障后，屏障后的load操作不能被重排序到屏障前
public native void loadFence();
//内存屏障，禁止store操作重排序。屏障前的store操作不能被重排序到屏障后，屏障后的store操作不能被重排序到屏障前
public native void storeFence();
//内存屏障，禁止load、store操作重排序
public native void fullFence();
```

#### 典型应用

在 Java 8 中引入了一种锁的新机制——StampedLock，它可以看成是读写锁的一个改进版本。StampedLock 提供了一种乐观读锁的实现，这种乐观读锁类似于无锁的操作，完全不会阻塞写线程获取写锁，从而缓解读多写少时写线程“饥饿”现象。由于 StampedLock 提供的乐观读锁不阻塞写线程获取读锁，当线程共享变量从主内存 load 到线程工作内存时，会存在数据不一致问题，所以当使用 StampedLock 的乐观读锁时，需要遵从如下图用例中使用的模式来确保数据的一致性。

![[StampedLock.png]]

如上图用例所示计算坐标点 Point 对象，包含点移动方法 move 及计算此点到原点的距离的方法 distanceFromOrigin。在方法 distanceFromOrigin 中，首先，通过 tryOptimisticRead 方法获取乐观读标记；然后从主内存中加载点的坐标值 (x,y)；而后通过 StampedLock 的 validate 方法校验锁状态，判断坐标点(x,y)从主内存加载到线程工作内存过程中，主内存的值是否已被其他线程通过 move 方法修改，如果 validate 返回值为 true，证明(x, y)的值未被修改，可参与后续计算；否则，需加悲观读锁，再次从主内存加载(x,y)的最新值，然后再进行距离计算。其中，校验锁状态这步操作至关重要，需要判断锁状态是否发生改变，从而判断之前 copy 到线程工作内存中的值是否与主内存的值存在不一致。

下图为 StampedLock.validate 方法的源码实现，通过锁标记与相关常量进行位运算、比较来校验锁状态，在校验逻辑之前，会通过 Unsafe 的 loadFence 方法加入一个 load 内存屏障，目的是避免上图用例中步骤 ② 和 StampedLock.validate 中锁状态校验运算发生重排序导致锁状态校验不准确的问题。

![[StampedLock.validate.png]]

### 系统相关

这部分包含两个获取系统相关信息的方法。

```Java
//返回系统指针的大小。返回值为4（32位系统）或 8（64位系统）。
public native int addressSize();
//内存页的大小，此值为2的幂次方。
public native int pageSize();
```

#### 典型应用

如下图所示的代码片段，为 java.nio 下的工具类 Bits 中计算待申请内存所需内存页数量的静态方法，其依赖于 Unsafe 中 pageSize 方法获取系统内存页大小实现后续计算逻辑。

![[Bits.unsafe.pageSize.png]]

## 结语

本文对 Java 中的 sun.misc.Unsafe 的用法及应用场景进行了基本介绍，我们可以看到 Unsafe 提供了很多便捷、有趣的 API 方法。即便如此，由于 Unsafe 中包含大量自主操作内存的方法，如若使用不当，会对程序带来许多不可控的灾难。因此对它的使用我们需要慎之又慎。

## 参考资料

- [OpenJDK Unsafe source](http://hg.openjdk.java.net/jdk7/jdk7/jdk/file/9b8c96f96a0f/src/share/classes/sun/misc/Unsafe.java)
- [Java Magic. Part 4: sun.misc.Unsafe](http://mishadoff.com/blog/java-magic-part-4-sun-dot-misc-dot-unsafe)
- [JVM crashes at libjvm.so](https://www.zhihu.com/question/51132462)
- [Java 中神奇的双刃剑–Unsafe](https://www.cnblogs.com/throwable/p/9139947.html)
- [JVM 源码分析之堆外内存完全解读](http://lovestblog.cn/blog/2015/05/12/direct-buffer/)
- [堆外内存 之 DirectByteBuffer 详解](https://www.jianshu.com/p/007052ee3773)
- 《深入理解 Java 虚拟机（第 2 版）》
