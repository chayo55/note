### Dubbo简介
Dubbo是一款高性能、轻量级的开源Java RPC框架，它提供了**三大核心能力**：**RPC(Remote Procedure Calling)面向接口的远程方法调用**，**智能容错和负载均衡**，以及**服务自动注册和发现**。
### Dubbo的分层工作原理
#### 分层
大的范围分为三层：
业务层、由自己提供接口和实现业务。
RPC层、封装了整个RPC调用，负载均衡，集群容错，代理
Remoting层、封装了网络传输协议，和数据转换。
![[Dubbo分层.jpg]]
#### 工作原理
1. 服务启动时，根据配置信息，`provider`和`consumer`连接到`register`，`provider`向注册中心注册提供的服务；`consumer`向注册中心订阅服务；
2. `consumer`会将register的服务信息列表同步到本地，方便调用，同时在`provider`有变动时，会收到register的广播推送。
3. `consumer`生成代理对象，根据负载均衡策略，选择一台`provider`，同时向`monitor`记录接口的调用次数和时间信息。
4. 通过代理对象，发起服务调用。
5. `provider`收到请求后，对数据进行反序列化，然后通过代理调用具体的接口实现。

通过代理对象通信，是为了实现接口的透明代理，封装调用细节，让用户像调用本地方法一样调用，还可以通过代理实现一些其他功能，如：
调用的负载均衡策略、调用的数据统计监控、调用超时，失败时的降级容错机制、过滤，缓存操作。
#### Dubbo中provider的服务导出
大致可分为三个部分，
第一部分是前置工作，主要用于检查参数，组装 URL。
第二部分是导出服务，包含导出服务到本地 (JVM)，和导出服务到远程两个过程。
第三部分是向注册中心注册服务，用于服务发现。
容器启动时，按照配置信息初始化好之后，触发`ContextRefreshEvent`事件，开始注册服务；
`ProxyFactory`获取到`invoker`，`invoker`包含了**需要执行的方法**的对象信息和具体的**URL地址**；再通过`DubboProtocol`的实现把包装后的`invoker`转换成`exporter`，然后启动服务器server，监听端口；最后`RegisterProtocol`保存`URL`和`exporter`的映射关系，同时注册到**register注册中心**

---
### Dubbo的负载均衡策略
- **加权随机**(按权重比分配区间，然后随机落到某个区间)
- **一致性hash**(一致性hash算法)
- **最小活跃数**(挑选当前最小活跃数(也就是负载最小)的服务器)
- **加权轮询**(按权重比分配服务次数)
```java
@RestController
public class UserController {
	
	//利用dubbo的方式为接口创建代理对象 利用rpc调用
	//@Reference(loadbalance = "random")			//加权随机  默认策略  
	//@Reference(loadbalance = "roundrobin")		//加权轮询
	//@Reference(loadbalance = "consistenthash")	//一致性hash  消费者绑定服务器提供者
	@Reference(loadbalance = "leastactive")			//最小活跃数
	private UserService userService; 

}
```
---
### Dubbo的容错机制
1. **Dubbo内置容错机制**
[官网介绍](https://dubbo.apache.org/zh-cn/blog/dubbo-cluster-error-handling.html)

|策略名称|	优点|	缺点	| 主要应用场景|
|---|---|---|---|
|Failover	|对调用者屏蔽调用失败的信息	|增加RT，额外资源开销，资源浪费|	对调用rt不敏感的场景|
|Failfast|	业务快速感知失败状态进行自主决策|	产生较多报错的信息|	非幂等性操作，需要快速感知失败的场景|
|Failsafe|	即使失败了也不会影响核心流程|	对于失败的信息不敏感，需要额外的监控|	旁路系统，失败不影响核心流程正确性的场景|
|Failback|	失败自动异步重试|	重试任务可能堆积|	对于实时性要求不高，且不需要返回值的一些异步操作
|Forking|	并行发起多个调用，降低失败概率	|消耗额外的机器资源，需要确保操作幂等性	|资源充足，且对于失败的容忍度较低，实时性要求高的场景||
|Broadcast	|支持对所有的服务提供者进行操作|	资源消耗很大|	通知所有提供者更新缓存或日志等本地资源信息|

2. **自定义容错策略**
如果上述内置的容错策略无法满足你的需求，还可以通过扩展的方式来实现自定义容错策略。
---
### Dubbo的SPI机制(Service Provider Interface)
***本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类，这样可以在运行时，动态为接口替换实现类。***

---

### Dubbo的配置
- provider服务者
	```yml
	#关于Dubbo配置   
	dubbo:
	  scan:
		basePackages: com.jd    #指定dubbo的包路径 扫描dubbo注解
	  application:              #应用名称
		name: provider-user     #一个接口对应一个服务名称   一个接口可以有多个实现
	  registry:  #注册中心 用户获取数据从机中获取 主机只负责监控整个集群 实现数据同步
		address: zookeeper://192.168.126.129:2181?backup=192.168.126.129:2182,192.168.126.129:2183
	  protocol:  #指定协议
		name: dubbo  #使用dubbo协议(tcp-ip)  web-controller直接调用sso-Service
		port: 20880  #每一个服务都有自己特定的端口 不能重复.
	```
- consumer
	```yml
	dubbo:
	  scan:
		basePackages: com.jd
	  application:
		name: consumer-user   #定义消费者名称
	  registry:               #注册中心地址
		address: zookeeper://192.168.126.129:2181?backup=192.168.126.129:2182,192.168.126.129:2183
	```
