# Java基础

### Java中精度问题
* BigDecimal
```BigDecimal(Double)```存在精度丢失的问题，
应该使用```BigDecimal(String)```，
或者```BigDecimal.ValueOf()```，内部执行了```Double.toString()```方法，按double实际能表达的精度 对尾数进行了截取
```Java
BigDecimal temp1 = new BigDecimal("0.1");
BigDecimal temp2 = BigDecimal.ValueOf(0.1);
```
* 浮点数
浮点数之间的等值判断，
基本数据类型不能用\==来比较，
包装数据类型不能用equals 来判断
```Java
//反例
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
if (a == b){
	//预期进入此代码块，
	//但事实a==b 为false
}

Float x = Float.valueOf(a);
Float y = Float.valueOf(b);
if (x.equals(y)){
	//预期进入此代码块，
	//但事实equals的结果为false
}

//正例
//(1)指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
float diff = 1e-6f;
if (Math.abs(a - b) < diff){
	System.out.println("true");
}

//(2)使用BigDecimal来定义值，再进行浮点数的运算操作
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");
BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);
if (x.equals(y)){
	System.out.println("true");
	}
}
```

---

### 泛型
* 泛型：&emsp;&emsp;泛型是一种编译时的类型，仅在编译阶段有效，运行时都为Object类型
	- 泛型定义时常用方式有三种(可参考`List<E>,Map<K,V>`等接口定义): 
		1. 泛型类：`class 类名<泛型,…>{}`
		2. 泛型接口：`interface 接口名<泛型,…>{}`
		3. 泛型方法:  `<泛型> 方法返回值类型  方法名(形参){}`
	- 为什么要使用泛型？
		1. 在编译阶段解决一些运行时需要关注的问题，比如强转
		2. 提供编程时灵活性
	
---
	
### String、 StringBuilder、 StringBuffer
#### __String__    
被 __final修饰__， 每次新建String对象，会在[[JVM内存区域划分#方法区 Method Area|方法区]]中的 __String常量池__ 查询是否存在，没有才新建对象。
###### String常量池  
在jdk1.7以前，常量池在永久代，
`JVM HotSpot在jdk1.7开始取消永久代，jdk1.8就完全取消永久代，引入元空间(Metaspace)`
从1.7开始，String常量池放在堆(Heap)中，
**经典案例：**  

```Java
	public static void main(String[] args) {
		String s = new String("1");	//新建了2个对象 [1、引用s指向的对象，2、常量池中对象 "1"]
		
		s.intern();		//jdk文档说明，
						// native intern()方法用以检查常量池中是否有equals相等的常量了，
						// 有则直接返回引用。没有则在常量池新建一个对象，再返回引用
		String s2 = "1";	// 以上代码一共新建了2个对象 [1、引用s指向的对象，2、常量池中对象 "1"]
							// 引用s2是直接指向了常量池中的"1"
		System.out.println(s == s2);   // 所以false

		String s3 = new String("1") + new String("1");
		s3.intern();
		String s4 = "11";
		System.out.println(s3 == s4);
	}
	//JDK1.7以前：		 false	 false
	//JDK1.7和以后：    false	true
```
		`将各  .intern()  方法下移`
```Java
	public static void main(String[] args) {
		String s = new String("1");
		String s2 = "1";
		s.intern();
		System.out.println(s == s2);

		String s3 = new String("1") + new String("1");
		String s4 = "11";
		s3.intern();
		System.out.println(s3 == s4);
	}
	//JDK1.7以前：		 false	 false
	//JDK1.7和以后：    false	false
```
  [[参考链接#深入解析String intern https tech meituan com 2014 03 06 in-depth-understanding-string-intern html|深入解析String intern]]  


#### __StringBuilder__    
继承自`AbstractStringBuilder`，内部实现：`和ArrayList类似`，维护了一个数组，通过将源对象转为`char[]`数组，进行append()操作时，检查数组大小是否足够，不够需要通过`Arrays.copyOf()`扩容。   

#### __StringBuffer__    
同样也继承自`AbstractStringBuilder`，内部实现同`StringBuilder`一样，在`append()`方法上加`synchronized`，以保证线程安全。

---

### Java中四大引用
强引用

一直活着：类似 “Object obj=new Object()” 这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象实例。

弱引用

回收就会死亡：被弱引用关联的对象实例只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象实例。在 JDK 1.2之后，提供了 WeakReference 类来实现弱引用。

软引用

有一次活的机会：软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象实例列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。在 JDK 1.2之后，提供了 SoftReference 类来实现软引用。

虚引用

也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象实例是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象实例被收集器回收时收到一个系统通知。在 JDK 1.2之后，提供了 PhantomReference 类来实现虚引用。

---

### Java中位运算符

1. 位与 &
两个整数值的32位，每一位和每一位求与。
***两位都是1与得的结果是1；只要有0结果就是0。***

	```
	00000000 00000000 00000000 01101001
	00000000 00000000 00000000 00100011 &
	----------------------------------------
	00000000 00000000 00000000 00100001
	```

2. 位或 |
***只要有1结果就是1；两位都是0结果是0。***

	```
	00000000 00000000 00000000 01101001
	00000000 00000000 00000000 00100011 |
	----------------------------------------
	00000000 00000000 00000000 01101011
	```

3. 异或 ^
异或运算是“找不同”
***不同是1；相同是0。***

	```
	00000000 00000000 00000000 01101001
	00000000 00000000 00000000 00100011 ^
	----------------------------------------
	00000000 00000000 00000000 01001010
	```
***异或还有个特点，就是对同一个值异或两次会得到原值。***
试着用上面异或的结果再对第二个值求一次异或，这样可以还原成第一个值。


4. 求反 ~
***1变0；0变1。***
求反运算只有一个运算项

	```
	00000000 00000000 00000000 01101001 ~
	----------------------------------------
	11111111 11111111 11111111 10010110
	```

---


### 类的加载
[https://zhuanlan.zhihu.com/p/72683366](https://zhuanlan.zhihu.com/p/72683366)
加载、验证、准备、解析、初始化、使用、卸载
加载：将字节码文件加载到JVM内存
验证：验证字节码文件是否符合规范，代码逻辑验证是否正确
准备：
分析一个类的执行顺序大概可以按照如下步骤：

- **确定类变量的初始值**。在类加载的准备阶段，JVM 会为类变量初始化零值，这时候类变量会有一个初始的零值。如果是被 final 修饰的类变量，则直接会被初始成用户想要的值。


- **初始化入口方法**。当进入类加载的初始化阶段后，JVM 会寻找整个 main 方法入口，从而初始化 main 方法所在的整个类。当需要对一个类进行初始化时，会首先初始化类构造器（），之后初始化对象构造器（）。


- **初始化类构造器**。JVM 会按顺序收集类变量的赋值语句、静态代码块，最终组成类构造器由 JVM 执行。


- **初始化对象构造器**。JVM 会按照收集成员变量的赋值语句、普通代码块，最后收集构造方法，将它们组成对象构造器，最终由 JVM 执行。

如果在初始化 main 方法所在类的时候遇到了其他类的初始化，那么就先加载对应的类，加载完成之后返回。如此反复循环，最终返回 main 方法所在类。