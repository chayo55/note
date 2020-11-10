## shiro是什么？
是Apache旗下一个开源安全框架
[http://shiro.apache.org/](http://shiro.apache.org/)
核心对象,主要包括:认证管理对象,授权管理对象,会话管理对象,缓存管理对象,加密管理对象以及Realm管理对象(领域对象:负责处理认证和授权领域的数据访问题)等
![[shiro核心对象.png]]
Subject :主体对象，负责提交用户认证和授权信息。
SecurityManager：安全管理器，负责认证，授权等业务实现。
Realm：领域对象，负责从数据层获取业务数据。

---
## shiro配置
### 手动配置

在配置类SpringShiroConfig中把`SecurityManager`和`ShiroFilterFactoryBean`对象交给spring
```Java
@Configuration
public class SpringShiroConfig {

	/** 在Shiro配置类中添加SecurityManager配置org.apache.shiro.mgt.SecurityManager*/
    @Bean
    public SecurityManager securityManager (Realm realm) {
		DefaultWebSecurityManager webSecurityManager = 
		new DefaultWebSecurityManager();
        webSecurityManager.setRealm(realm);
        return webSecurityManager;
	}
	
	/** 在Shiro配置类中添加ShiroFilterFactoryBean对象的配置。通过此对象设置资源匿名访问、认证访问。 */
	@Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean (SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = 
		new ShiroFilterFactoryBean();
		//配置拦截规则
		//放行静态资源、登录路径、首页、等，视业务而定
		
		return shiroFilterFactoryBean;
	}
	/** 配置advisor对象,shiro框架底层会通过此对象的matchs方法返回值(类似切入点)决定是否创建代理对象,进行权限控制。 */
	@Bean
	public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor (
	    		    SecurityManager securityManager) {
		AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
		advisor.setSecurityManager(securityManager);
		return advisor;
	}

```
---
### 认证(Authentication)流程分析如下:
1. 系统调用`subject`的`login()`方法将用户信息提交给`SecurityManager`
2. `SecurityManager`将认证操作委托给认证器对象`Authenticator`
3. `Authenticator`将用户输入的身份信息传递给`Realm`。 
4. `Realm`访问数据库获取用户信息然后对信息进行封装并返回。
5. `Authenticator `对`realm`返回的信息进行身份认证。
![[shiro认证Authentication流程分析.png]]
---
### 授权(Authorization)流程分析如下：
1. 系统调用`subject`相关方法将用户信息(例如`isPermitted()`)递交给`SecurityManager`。
2. `SecurityManager`将权限检测操作委托给`Authorizer`对象。
3. `Authorizer`将用户信息委托给`realm`。
4. `Realm`访问数据库获取用户权限信息并封装。
5. `Authorizer`对用户授权信息进行判定。
![[shiro授权Authorization流程分析.png]]

### springboot自动配置

1. pom文件添加依赖
```Java
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>1.6.0</version>
</dependency>
```
2. 配置 配置类
```Java

	//配置realm
	@Bean
	public Realm realm() {
	  return new CustomRealm();
	}

	//配置filterChain
	@Bean
	public ShiroFilterChainDefinition shiroFilterChainDefinition() {
    	DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
    	chainDefinition.addPathDefinition("/**", "anon"); // all paths are managed via annotations
    
    	// or allow basic authentication, but NOT require it.
    	// chainDefinition.addPathDefinition("/**", "authcBasic[permissive]"); 
    	return chainDefinition;
	}
	
	//配置shiro内置cache	
	@Bean
	protected CacheManager cacheManager() {
		return new MemoryConstrainedCacheManager();
	}
```