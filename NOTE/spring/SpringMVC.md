### SpringMVC中请求流程：
1. springMVC请求路径图解
![[springMVC请求路径图解.jpg]]

### 定义Controller
- ~~实现Controller接口(不推荐)~~
- ~~实现HttpRequestHandler接口(不推荐)~~
- ~~实现Servlet接口(不推荐)~~
- 使用注解(主要使用)
@RequestMapping
@GetMapping@PostMapping
- 使用HandlerFunction (WebFlux)  

### 接受参数  
![[springMVC.png]]

### 参数类型转换


### 异常处理  

#### 注解原理：
注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。我们通过反射获取注解时，返回的是Java运行时生成的动态代理对象。通过代理对象调用自定义注解的方法，会最终调用AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池。
