==<JDK1.7以前>==
![[JVM内存区域划分.png]]

---
### 程序计数器(Program Counter Register)
- 用于存放 **下一条指令所在单元地址**
- 是**线程私有**的，每条线程都有一个独立的程序计数器
- 是JVM内存中唯一没有规定OutMemoryError的区域

---

### 虚拟机栈(Java virtual Machine Stacks)  
虚拟机栈是Java执行方法的内存模型。每个方法被执行的时候，都会创建一个栈帧，把栈帧压人栈，当方法正常返回或者抛出未捕获的异常时，栈帧就会出栈。
- 栈帧：栈帧存储方法的相关信息，包含局部变量数表、返回值、操作数栈、动态链接
- 是**线程私有**的，生命周期与线程相同
- 栈内存不足时，抛出StackOverflowError，OutofMemoryError

---

### 本地方法栈(Native Method Stack)
- 为Native方法服务
- **线程私有**
- 某些虚拟机，如HotSpot将其与栈合二为一
- 内存不足也会抛出，StackOverflowError，OutofMemoryError

---

### 堆(Heap)
- 用于存放 几乎所有**对象实例**，**数组**。
- 是**所有线程共享**的内存区域
- 内存不足抛出，OutOfMemoryError
- 堆内存可以细分为 **`新生代(Young Generation)`**、**`老年代(Old Generation)`**，新生代又能细分为**`伊甸区(Eden)`**和两个幸存者区(**`From Survivor`，`To Survivor`**)。如下图：
![[新生代老年代.png]]
图中 **`永久代(Permanent Generation)`**是Hotspot虚拟机特有的概念，是Hotspot把GC分代扩展至了方法区，是对**[[JVM内存区域划分#方法区 Method Area|方法区]]**的一种实现，别的JVM都没有。

---
### 方法区(Method Area)
- 用来存储，**类信息、常量、静态变量、String对象**等
- 是**所有线程共享**的内存区域
- HotSpot在JDK1.7将**String常量池**移到了 java heap，
同时还有，字面量(interned strings)，类的静态变量(class statics)都转移到了 java heap。符号引用(Symbols)转移到Native Memory，
- jdk1.8中就不再有方法区了，Hotspot对方法区的实现为，**元空间Metaspace**
元空间不再与堆连续，而且是存在于本地内存（Native memory),所以不会再有类似永久代中的`java.lang.OutOfMemoryError: PermGen space`错误.
==<JDK1.8后>==
![[jdk1.8后内存区域图.jpg]]