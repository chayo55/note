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

