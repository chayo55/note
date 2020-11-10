### 什么是AOP？
AOP 面向切面编程，是一种软件设计思想，是对OOP面向对象编程的补充，通过预编译(lombok)的方式，或运行期动态代理(jdk,cglib)的方式，对程序进行扩展或添加额外功能。
### Spring AOP
1. Spring 原生实现方式<br>Spring AOP原生方式实现的核心有三大部分构成，分别是：
	1. **JDK代理**
	只能代理实现了接口的类，**代理类**通过和**目标类**实现共同接口  
	2. **CGLIB代理**
	可以代理未实现接口的类，通过继承**目标类**产生的子类 来拦截目标类的方法调用，所以，CGLIB代理的方式不能代理final修饰的类和方法  
	3. **org.aopalliance包下的拦截体系。**
		
	`就二者的效率来说，大部分情况都是 JDK 动态代理更优秀，随着 JDK 版本的升级，这个优势更加明显。    `
	
2. AspectJ 方式<br>AOP相关术语
	- Aspect(切面)：横切面对象，在springAOP中用@Aspect声明
	切面优先级：注解 @Order(1)  数字越小优先级越高  

	- JoinPoint(连接点，切入点)：连接点是程序执行过程中某个特定的点，一般指被拦截到的的方法。
		**切入点表达式：**  
		*粗粒度：*
		- bean: `("bean("*ServiceImpl")")`
		- winthin: `("within("aop.service.*")")`  
		
		*细粒度：*
		- execution: `("execution("void listUserInfo(String)")")`
		- annotation: `(annotation(anno.Transactional))`  

	- Advice(通知)：在切面的某个特定连接点**要扩展的功能**    
		基于AspectJ框架标准，spring中定义了五种类型的通知：
		- @Before` 目标方法执行之前执行 `
		- @Around
		- @After`目标方法结束之前(return或throw之前)执行 `
		- @AfterReturning ` after之后程序没有异常执行此通知 `
		- @AfterThrowing`after之后程序出现异常则执行此通知 `
		![[SpringAOP五种通知执行顺序.png]]