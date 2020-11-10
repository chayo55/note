### 接口中可以写默认方法和静态方法
```Java
interface Human {
	// 默认方法
	default void eat() {
		System.out.println("eat...");
	}
	// 静态方法
	public static void stand() {
		System.out.println("stand...");
	}
}
```
**应用场景**
在一个接口A下的一些了实现类中，需求需要在其中两三个实现类扩展方法，JDK8以前的解决方法：添加一个抽象类B，继承接口A，将需要扩展的方法写在抽象类B中，这两三个目标实现类再继承抽象类B。
JDK8后，在接口中可以写默认方法，以上场景只需要在接口中添加默认方法，在到目标实现类中重写该方法即可。

**接口默认方法的冲突问题**
如果多继承的情况下，子类会不清楚继承哪个接口的方法，所以需要重写
```Java
interface Human {
	default void eat() {
		System.out.println("human eat");
	}
}
interface Animal {
	default void eat() {
		System.out.println("Animal eat");
	}
}
interface A extends Human,Animal {
	@Override
	default void eat() {
		System.out.println("A eat");
	}
}
```

--- 


### Stream
