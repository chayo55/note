Spring架构
![[spring架构.png]]

组成 Spring 框架的每个模块（或组件）都可以单独存在，或者与其他一个或多个模块联合实现。每个模块的功能如下：

|      | 模块                | 说明                                                         |
| ---- | ------------------- | ------------------------------------------------------------ |
|      | 核心容器Spring Core | 核心容器，提供Spring框架的基本功能。核心容器的主要组件是BeanFactory，它是工厂模式的实现。BeanFactory 使用控制反转（IOC）模式，将应用程序的配置和依赖性规范与实际的应用程序代码分开。 |
|      | Spring Context      | Spring上下文，是一个配置文件，向 Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如 JNDI、EJB、电子邮件、国际化、校验和调度功能。 |
|      | Spring AOP          | 通过配置管理特性，Spring AOP 模块直接将面向切面的编程功能集成到了 Spring 框架中。可以很容易地使 Spring框架管理的任何对象支持AOP。Spring AOP模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，就可以将声明性事务管理集成到应用程序中。 |
|      | Spring DAO          | JDBC DAO 抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。 |
|      | Spring ORM          | Spring 框架插入了若干个 ORM 框架，从而提供了 ORM 的对象关系工具，其中包括JDO、Hibernate和iBatis SQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。 |
|      | Spring Web          | Web上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以Spring 框架支持与 Jakarta Struts的集成。Web模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。 |
|      | Spring MVC框架      | MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。 |


## spring容器启动流程
[Spring源码系列之容器启动流程](https://mp.weixin.qq.com/s/q6zs7xRjpcB4YxLw6w477w)
### 基础概念
1. `Spring`会将所有交由`Spring`管理的类，扫描其`class`文件，将其解析成`BeanDefinition`，在`BeanDefinition`中会描述类的信息，例如:这个类是否是单例的，`Bean`的类型，是否是懒加载，依赖哪些类，自动装配的模型。`Spring`创建对象时，就是根据`BeanDefinition`中的信息来创建`Bean`。

2. `Spring`容器在本文可以简单理解为`DefaultListableBeanFactory`,它是`BeanFactory`的实现类，这个类有几个非常重要的属性：`beanDefinitionMap`是一个`map`，用来存放`bean`所对应的`BeanDefinition`；`beanDefinitionNames`是一个`List`集合，用来存放所有`bean`的`name`；`singletonObjects`是一个`Map`，用来存放所有创建好的单例`Bean`。

3. `Spring`中有很多后置处理器，但最终可以分为两种，一种是`BeanFactoryPostProcessor`，一种是`BeanPostProcessor`。前者的用途是用来干预`BeanFactory`的创建过程，后者是用来干预`Bean`的创建过程。后置处理器的作用十分重要，`bean`的创建以及`AOP`的实现全部依赖后置处理器。

### 大概流程
通过`AnnotationConfigApplicationContext`的有参构造方法入手，三个方法：`this(); ``register(componentClasses);` `refresh();`
在`this()`中初始化了一个`BeanFactory`，即`DefaultListableBeanFactory`；然后向容器中添加了7个内置的`bean`，其中就包括`ConfigurationClassPostProcessor`。

在`refresh()`方法中，又重点分析了`invokeBeanFactoryPostProcessor()`方法和`finishBeanFactoryInitialization()`方法。

在`invokeBeanFactoryPostProcessor()`方法中，通过`ConfigurationClassPostProcessor`类扫描出了所有交给`Spring`管理的类，并将`class`文件解析成对应的`BeanDefinition`。

在`finishBeanFactoryInitialization()`方法中，完成了非懒加载的单例`Bean`的实例化和初始化操作，主要流程为`getBean()` ——>`doGetBean()`——>`createBean()`——>`doCreateBean()`。在`bean`的创建过程中，一共出现了8次`BeanPostProcessor`的执行，在这些后置处理器的执行过程中，完成了`AOP`的实现、`bean`的自动装配、属性赋值等操作。

[通过源码看Bean的创建过程](https://mp.weixin.qq.com/s/WwjicbYtcjRNDgj2bRuOoQ)
Spring中单例Bean的生命周期的图示：
![[spring中bean的生命周期图.jpg]]
