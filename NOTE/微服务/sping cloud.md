# Spring Cloud
![[springcloud技术组成.png]]
Spring Cloud 是一个工具集，集成了多种工具，来解决微服务中的各种问题
微服务的整体解决方案，微服务全家桶
- 服务发现
- 远程调用
- 负载均衡
- 系统容错 - 降级、熔断
- API网关
- 配置中心
- 系统监控
- ....

 Spring Cloud 不是一个解决单一问题的框架

## Spring Cloud 和 [[Dubbo]]  区别
![[springcloud对比dubbo.png]]
- Dubbo 是远程调用工具
  - RPC调用，
  - java序列化
- Spring Cloud 集成工具，集成多种工具解决微服务中的所有问题
  - RestAPI 调用
  - Http

---

## eureka 注册中心
![[eureka.png]]
作用：服务注册和发现
- 提供者(Provider)

  向注册中心注册自己的地址

- 消费者(Consumer)

  从注册中心发现其他服务

#### eureka的运行机制

- 注册 - 一次次反复连接eureka，直到注册成功为止
- 拉取 - 每隔30秒拉取一次注册表，更新注册信息
- 心跳 - 每30秒发送一次心跳，3次收不到心跳eureka会删除这个服务
- 自我保护模式 - 特殊情况，由于网络不稳定15分钟内85%服务器出现心跳异常
  - 保护所有的注册信息不删除
  - 网络恢复后，可以自动退出保护模式
  - 开发测试期间，可以关闭保护模式

#### 搭建eureka服务
- eureka server 依赖
- yml配置
  - 关闭保护模式
  - 主机名（集群中区分每台服务器）
  - 针对单台服务器，不向自己注册，不从自己拉取
- 启动类
  - @EnableEurekaServer 触发eureka服务器的自动配置

### spring cloud 的远程调用
springboot 提供的远程调用工具 RestTemplate，类似 HttpClient
#### RestTemplate
RestTemplate 对http Rest API 调用做了高度封装，只需要调用一个方法就可以完成请求、响应、json转换
- getForObject(url,  转换的类型.class,   提交的参数数据)
- postForObject(url,  提交的协议体数据,  转换的类型.class)

---

## Ribbon
![[Ribbon.png]]
springcloud 提供的工具，对 RestTemplate 进行了增强封装，提供了负载均衡和重试的功能

### 负载均衡
1. 从 eureka 获得地址表
2. 使用多个地址来回调用
3. 拿到一个地址，使用 restTemplate 执行远程调用

添加负载均衡：

1. 添加 ribbon  依赖（eureka client中已经包含，不需要再单独添加）

2. 添加 @LoadBalanced 注解，对RestTemplate进行增强

3. 用 restTemplate 调用的地址，改成 service-id （注册中心注册的服务名）

   rt.getForObject("http://item-service/{1}", ......)

### ribbon重试

一种容错方式，调用远程服务失败(异常、超时)时，可以自动进行重试调用

添加重试：

1. 添加 spring-retry 依赖
2. 配置重试参数
   - MaxAutoRtries - 单台服务器的重试次数
   - MaxAutoRtriesNextServer - 更换服务器的次数
   - OkToRetryOnAllOperations - 是否对所有类型请求都进行重试，默认只对get重试
   - ConnectTimeout - 和远程服务建立连接的等待超时时长
   - ReadTimeout - 建立连接并发送请求后，等待响应的超时时长
   - 两个超时设置，不能在yml中配置，而是要在java代码中设置

---

## Hyxtrix
![[Hystrix.png]]
系统容错工具

- 降级
- 熔断

### 降级

调用远程服务失败（异常、超时、服务不存在），可以通过执行当前服务中的一段代码来向客户端发回响应

降级响应：

- 错误提示
- 返回缓存数据

快速失败

即使后台服务故障，也要让客户端尽快得到错误提示，而不能让客户端等待

添加降级

1. 添加 Hystrix 依赖

2. 启动类添加 @EnableCircuitBreaker

3. 添加降级代码

   在远程调用方法上添加

   @HystrixCommand(fallbackMethod="降级方法")

   完成降级方法，返回降级响应

### hystrix 超时

默认1秒超时，执行降级

如果配置了ribbon重试，重试还会继续执行，最终重试结果无效

Hystrix超时 >= Ribbon 总的超时时长

### hystrix  熔断

当请求量增大、出现过多错误，hystrix可以和后台服务断开连接（过热保护）

可以避免雪崩效应、故障传播

限流措施

流量过大时造成服务故障，可以断开服务，降低它的流量

在特定条件下会自动触发熔断

- 10秒内20次请求（必须首先满足）
- 50%出错，执行了降级代码

半开状态下可以自动恢复

- 断路器打开几秒后，进入半开状态，尝试发送请求，
  - 如果请求成功自动关闭断路器恢复正常
  - 如果请求失败，再保持打开状态几秒钟

---

## Hystrix Dashboard
![[Hystrix dashboard.png]]
### actuator
![[actuator.png]]
springboot 提供的项目监控工具，提供了多种项目的监控数据

- 健康状态
- 系统环境
- beans - spring容器中所有的对象
- mappings - spring mvc 所有映射的路径
- .....

hystrix在actuator中，添加了自己的监控数据

添加actuator
1. 添加 actuator 依赖
2. yml 配置暴露监控信息
   - m.e.w.e.i="*" - 暴露所有监控
   - m.e.w.e.i=["health",  "beans", "mappings"]
   - m.e.w.e.i=beans
3. http://xxxxxxxxx/actuator/

### 搭建  Hystrix Dashboard

1. 添加 Hystrix Dashboard 依赖
2. yml配置配置端口
3. 添加 @EnableHystrixDashboard

访问 http://xxxxxxx/hystrix

在输入框中填写要监控的数据路径
Hystrix监控仪表盘，监控 Hystrix 降级和熔断的错误信息
![[HystrixDashboard图表.png]]
![[Turbine Dashboard.png]]

---

## Turbine
hystrix dashboard 一次只能监控一个服务实例，使用 turbine 可以汇集监控信息，将聚合后的信息提供给 hystrix dashboard 来集中展示和监控
![[Turbine.png]]

---

## Feign
微服务应用中，ribbon 和 hystrix 总是同时出现，feign 整合了两者，并提供了声明式消费者客户端
![[Feign.png]]
### Feign 远程调用
Feign 提供了**声明式客户端**，只需要定义一个接口，就可以通过接口做远程调用。具体调用代码通过动态代理添加。

```java
// 调用商品服务的远程调用接口
// 通过注解配置3件事：调用哪个服务，调用什么路径，向这个路径提交什么参数
@FeignClient(name="item-service")
public interface ItemFeignClient {
    @GetMapping("/{orderId}")
    JsonResult<List<Item>> getItems(@PathVariable String orderId);
}
```


### Feign 集成 Ribbon

负载均衡、重试

Feign 默认启用了Ribbon的负载均衡和重试，0配置

默认重试参数：

- MaxAutoRetries=0
- MaxAutoRetriesNextServer=1
- ReadTimeout=1000



重试参数配置

```yml
# 对所有服务都有效
ribbon:
  MaxAutoRetries: 1
  
# 对 item-service 单独配置，对其他服务无效
item-service:
  ribbon: 
    MaxAutoRetries: 0
```



## Feign 集成 Hystrix

默认不启用 Hystrix，Feign不推荐启用hystrix（Feign用于服务间调用，Hystrix的降级，熔断功能，可能导致服务间调用混乱）
 如果有特殊需求要启用hystrix，首先做基础配置
1. 添加 hystrix 的完整依赖
2. 添加 @EnableCircuitBreaker
3. yml配置 feign.hystrix.enabled=true
### 降级
在声明式客户端接口的注解中，指定一个降级类

```java
@FeignClient(name="item-service", fallback=降级类.class)
public interface ItemFeignClient {
    ....
}
```

降级类要作为声明式客户端接口的子类来定义
### hystrix监控
用 actuator 暴露 hystrix.stream 监控端点
1. actuator 依赖
2. 暴露监控端点

```yml
m.e.w.e.i=hystrix.stream
```

3. http://localhost:3001/actuator/hystrix.stream
4. 通过调用后台服务，产生监控数据
### 熔断
10秒20次请求, 50%失败执行了降级代码

---

## Zuul API网关
![[Zuul.png]]
zuul API 网关，为微服务应用提供统一的对外访问接口。
zuul 还提供过滤器，对所有微服务提供统一的请求校验。
yml配置
```yml
# zuul 路由配置可以省略，缺省以服务 id 作为访问路径,但建议手动配置，避免注册信息不完整
zuul:
  routes:
    item-service: /item-service/**
    user-service: /user-service/**
    order-service: /order-service/**
```
主启动程序添加注解`@EnableZuulProxy`,`@EnableDiscoveryClient`
```java
@EnableZuulProxy
@EnableDiscoveryClient
@SpringBootApplication
public class Sp11ZuulApplication {
	public static void main(String[] args) {
		SpringApplication.run(Sp11ZuulApplication.class, args);
	}
}
```
### Zuul集成Ribbon
- **Zuul已经集成了Ribbon并**默认开启了负载均衡****
- **zuul + ribbon 重试**
	1. 添加依赖
	```pom
	<dependency>
		<groupId>org.springframework.retry</groupId>
		<artifactId>spring-retry</artifactId>
	</dependency>
	```
	2. 配置yml
	```yml
	zuul:
	  retryable: true
	ribbon:
	  ConnectTimeout: 1000
	  ReadTimeout: 1000
	  MaxAutoRetriesNextServer: 1
	  MaxAutoRetries: 1
	```
### zuul + hystrix 降级
创建降级类，并实现`FallbackProvider `接口，重写`getRoute()`,`fallbackResponse()` 方法,`ClientHttpResponse`响应类。
**getRoute() 方法中指定应用此降级类的服务id，星号或null值可以通配所有服务。**

zuul 已经包含 actuator 依赖，打开hystrix监控，只需暴露hystrix.stream 监控端点
```yml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

### zuul + turbine
```yml
turbine:
  app-config: order-service, zuul
  cluster-name-expression: new String("default")
```

### zuul请求过滤
![[zuul请求过滤.png]]
新建过滤类并继承`ZuulFilter`类
```java
@Component
public class AccessFilter extends ZuulFilter{
	@Override
	public boolean shouldFilter() {
	    //对指定的serviceid过滤，如果要过滤所有服务，直接返回 true
		RequestContext ctx = RequestContext.getCurrentContext();
		String serviceId = (String) ctx.get(FilterConstants.SERVICE_ID_KEY);
		if(serviceId.equals("item-service")) {
			return true;
		}
		return false;
	}
	@Override
	public Object run() throws ZuulException {
		RequestContext ctx = RequestContext.getCurrentContext();
		HttpServletRequest req = ctx.getRequest();
		String token = req.getParameter("token");
		if (token == null) {
			//此设置会阻止请求被路由到后台微服务
			ctx.setSendZuulResponse(false);
			//向客户端的响应
			ctx.setResponseStatusCode(200);
			ctx.setResponseBody(JsonResult.err().code(JsonResult.NOT_LOGIN).toString());
		}
		//zuul过滤器返回的数据设计为以后扩展使用，
		//目前该返回值没有被使用
		return null;
	}
	@Override
	public String filterType() {
		return FilterConstants.PRE_TYPE;
	}
	@Override
	public int filterOrder() {
	    //该过滤器顺序要 > 5，才能得到 serviceid
		return FilterConstants.PRE_DECORATION_FILTER_ORDER+1;
	}
}
```

---

## Config配置中心
![[Config配置中心.png]]
yml 配置文件保存到 git 服务器，例如 github.com 或 gitee.com
微服务启动时，从服务器获取配置文件
- 搭建配置服务中心
引入依赖
```pom
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
yml配置
```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/你的个人路径/sp-config
          searchPaths: config
          #username: your-username
          #password: your-password
```
主启动类添加注解`@EnableConfigServer`
- 客户端配置
引入依赖
```pom
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```
yml配置
```yml
spring: 
  cloud:
    config:
      discovery:
        enabled: true
        service-id: config-server
      name: item-service
      profile: dev
```
服务中心和客户端，都需要eureka依赖，启动类添加`@EnableDiscoveryClient`

---

## BUS消息总线
添加[[RabbitMQ]]依赖，bus依赖
```pom
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-bus</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```
服务中心及客户端的配置yml文件
```yml
spring:
  rabbitmq:
    host: 192.168.126.140
    username: admin
    password: admin
```
config-server服务中心 暴露bus-refresh 刷新端点
```yml
management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```
post请求发送refresh，请求刷新端点发布刷新消息
`http://config-server-ip:port/actuator/bus-refresh`

刷新指定的微服务
`http://config-server-ip:port/actuator/bus-refresh/user-service:8001`

--- 

## Sleuth + Zipkin
![[Sleuth+Zipkin.png]]
随着系统规模越来越大，微服务之间调用关系变得错综复杂，一条调用链路中可能调用多个微服务，任何一个微服务不可用都可能造整个调用过程失败

spring cloud sleuth 可以跟踪调用链路，分析链路中每个节点的执行情况

### 微服务中添加 spring cloud sleuth 
添加依赖
```pom
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```
微服务的控制台日志中，可以看到以下信息：
`[服务id,请求id,span id,是否发送到zipkin]`

- id：请求到达第一个微服务时生成一个请求id，该id在调用链路中会一直向后面的微服务传递
- an id：链路中每一步微服务调用，都生成一个新的id

### Zipkin
zipkin 可以收集链路跟踪数据，提供可视化的链路分析.
微服务中配置Zipkin
引入依赖
```pom
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```
默认 10% 的链路数据会被发送到 zipkin 服务。可以配置修改抽样比例:
```yml
spring:
  sleuth:
    sampler:
      probability: 0.1
```
启动Zipkin
`java -jar zipkin-server-2.12.9-exec.jar --zipkin.collector.rabbitmq.uri=amqp://admin:admin@192.168.64.140:5672`
![[zipkin图标界面.png]]
![[zipkin依赖关系图.png]]