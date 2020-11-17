# SPRINGBOOT

## springboot

1. ### springboot有哪些关键特性

	- 起步依赖
	- 自动配置
	- 健康检查
	- 嵌入式服务
---
2. ### springboot项目启动时发生了什么？

	- springboot启动过程分析
	![[springboot启动过程分析.png]]
	- 启动后，会找到@SpringBootApplication描述的启动类，类中会定义一个main方法，main方法运行时会读取配置文件，加载资源(资源可以理解为class文件)配置文件
		- 配置文件
			- SpringBoot官方定义的配置文件
			- 自己定义的配置文件
		- 加载的资源
			- jdk的类文件
			- spring类文件
			- 自己定义的类文件
			- spring中的资源初始化
				- 构建对象
				- 基于对象存储数据(例如配置信息，默认值)
---
3. ### spring管理Bean对象
- SpringBoot启动后Bean的实例化过程
![[SpringBoot启动后Bean的实例化过程.png]]
- spring管理Bean对象有什么优势？
	1. 高效，低耗的应用，为管理的Bean提供:
		1. 懒加载策略
		```
		@Lazy 设置延迟加载（对象暂时用不到，则无需加载和实例化,spring中默认为true)
		```
		2. 作用域
		```
		@Scope 设置作用域 
		sington 单例（使用频繁时,会交给spring管理生命周期)
		prototype 多例（使用不频繁时，spring负责实例化,不负责销毁，
		所以@PreDestroy 此方法不会执行）
		```
		3. 生命周期方法（每个对象都有生命周期）
		```
		设置生命周期方法（更好实现对象的初始化和资源销毁)
		@PostConstruct(对象构造方法之后执行) 
		@PreDestroy(对象销毁之前执行)
		（@ProstConstruct @PreDestroy属于java的注解，不属于spring）
		```
	2. 可维护性：
				`降低对象与对象间的耦合`
---
## hikariCP
1. 请问这个获取连接的过程你了解吗?

	- 1. 第一次获取连接时首先会检测连接池是否存在,假如不存在则创建并初始化池.
	- 2. 然后基于Driver对象创建与数据库的链接,并将链接放入池中.
	- 3. 最后从池中返回用户需要的链接
---
## spring

### 事务 [[Transactional|Transactional]]
![[Transactional#Spring中事务管理]]

---
## [[SpringMVC]]
![[SpringMVC]]
## mybatis
---
1. spring整合mybatis
	- POM文件引入mybatis
	- 在properties文件，配置mybatis,  mapper-location等

2. 旧版本指定参数需要@Param注解
`int deleteById(@Param("id") Integer id);`
`jdk8以前，无法取到参数名，内部参数名都为args，所以需要@Param注解指定 需要给到sql的参数名`




CommandLineRunner 此接口定义了一种规范，可以在spring框架初始化以后执行一些逻辑。
实现ComandLineRunner重写run方法，可以获取spring容器中的一些初始化资源[[SpringBoot#springboot]]


