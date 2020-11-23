### 定义
事务(Transaction)是一个业务,是一个不可分割的逻辑工作单元，基于事务可以更好的保证业务的正确性。

---

### 事务特性 ACID
1. **Atomic**　　　　==原子性==　事务中的操作不可分割，全部成功或者全部失败
2. **Consistency**　 ==一致性==　事务前后的状态一致性
3. **Islostion**　　 　==隔离性==　事务与事务之间是相互隔离的
	- 隔离级别：
		- `read uncommit`  
		读未提交，可能会读到其他事务未提交的数据，也叫做脏读。
		
		- `read commit`  
		读已提交，两次读取结果不一致，叫做不可重复读。
		
		- `repeatable read`  
		可重复复读，这是[[MYSQL]]的默认级别，就是每次读取结果都一样，但是有可能产生幻读(当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户再读取该范围的数据行时，会发现有新的“幻影” 行)。
		
		- `serializable`  
		串行，一般是不会使用的，他会给每一行读取的数据加锁，会导致大量超时和[[锁]]竞争的问题。
		
		>
		>|  四种情况对应四种隔离级别   |   |
		>|  ----  | ----  |
		>|脏写|读未提交 read uncommit|
		>|脏读|读已提交 read commit|
		>|不可重复读|可重复读 read uncommit|
		>|幻读|串行化 serializable|
		>- ` 脏写`：是指事务将其他事务的修改回滚了。
		>`eg`:事务1 对数据data修改为data1，但还未提交，事务2 开始，对数据修改为data2，并提交事务。事务1 因为某些原因，进行事务回滚，将数据回滚为data。这种事务2 修改数据并提交事务后，却被事务1 回滚了的现象称为 脏写。
		>`解决方式`：在数据库中对数据data加上写锁，实现方式就是设置隔离级别为 ***读未提交read uncommit***   
		><br>
		>- `脏读`：是指事务读取到了其他事务未提交的数据。
		>`eg`：事务1对数据data进行修改为data1，但还未提交，事务2开始，对数据查询得到事务1 还未提交的数据data1， 事务1 因为某些原因，进行事务回滚，将数据回滚为data。这种事务2 读取到事务1 还未提交的数据的现象称为 脏读。
		>`解决方式`：设置隔离级别为 ***读已提交 read commit***
		><br>
		>- `不可重复读`：是指同一事务中，进行两次查询，前后查询结果不一致的情况。
		>`eg`：事务1 开启事务，查询数据得到结果data，但还未提交，事务2 开始，对数据进行修改为data2，事务2提交。事务1 中再进行同样的查询，得到事务2修改后的结果data2，导致前后两次查询结果data≠data2不一致的现象称为不可重复读。
		>`解决方式`：设置隔离级别为 ***可重复读 repeatable read***
		><br>
		>- `幻读`：是指同一事务中，进行两次查询，后一次查询读到了前一次查询没有的数据。
		>`eg`：事务1开启事务，查询数据得到data，但还未提交，事务2开始，对数据进行增加 add data2，事务2 提交。事务1 中再进行查询，得到结果data data2，导致前后两次查询，后一次查询比之前查询多读到了数据data2，这种现象称为**幻读**。
		>`解决方式`：设置隔离级别为 ***串行化 serializable***
		
		
4. **Durability**　　  ==持久性==　事务一旦提交，数据要持久保存
---
### 那ACID靠什么保证的呢？
==*Atomic*==. 　原子性由undo log日志保证，它记录了需要回滚的日志信息，事务回滚时撤销已经执行成功的sql

==*Consistency*==. 　一致性一般由代码层面来保证

==*Islostion*==. 　隔离性由[[Transactional#MVCC Multi-Version Concurrency Control 多版本并发控制|MVCC]]来保证

==*Durability*==. 　持久性由内存+redo log来保证，mysql修改数据同时在内存和redo log记录这次操作，事务提交的时候通过redo log刷盘，宕机的时候可以从redo log恢复

---

### MVCC(Multi-Version Concurrency  Control)多版本并发控制
MVCC的实现是通过保存数据在某一个时间点快照来实现的。也就是说不管实现时间多长，每个事务看到的数据都是一致的。

分为乐观（optimistic）并发控制和悲观（pressimistic）并发控制。

**MVCC是如何工作的：**
InnoDB的MVCC是通过在每行记录后面保存两个隐藏的列来实现。这两个列一个保存了行的**创建版本号**，一个保存行的**删除版本号**。
每一个事务在启动的时候，都有一个唯一的递增的版本号。
- 插入操作 ： 记录的创建版本号就是事务版本号。 
- 更新操作：采用的是先标记旧的那行记录为已删除，并且删除版本号是事务版本号，然后插入一行新的记录的方式。
- 删除操作：就把事务版本号作为删除版本号。
- 查询操作：在查询时要符合以下两个条件的记录才能被事务查询出来： 
	1) 删除版本号 大于 当前事务版本号，就是说删除操作是在当前事务启动之后做的。 
	2) 创建版本号 小于或者等于 当前事务版本号 ，就是说记录创建是在事务中（等于的情况）或者事务启动之前。

MVCC只在`COMMITTED READ`（读提交）和`REPEATABLE READ`（可重复读）两种隔离级别下工作。`Serializable`则会对所有读取的行加锁。

**优点**是通过版本号来减少锁的争用。
**缺点**是每行记录都需要额外的存储空间，需要做更多的行维护和检查工作。

---

### Spring中事务管理
在[[SpringBoot]]项目中,其内部提供了事务的自动配置，当我们在项目中添加了指定依赖spring-boot-starter-jdbc时，框架会自动为我们的项目注入事务管理器对象，最常用的为`DataSourceTransactionManager`对象。
```Java
@Transactional(timeout = 30,		//设置事务超时时间 默认-1永不超时
               readOnly = false,	//设置事务是否只读 默认false
               isolation = Isolation.READ_COMMITTED,	//设置事务隔离级别 默认DEFAULT 表示使用底层数据库的默认隔离级别
               rollbackFor = Throwable.class,	//用于指定能够触发事务回滚的异常类型，no-rollback-for 指定的异常类型，不回滚事务。
               propagation = Propagation.REQUIRED)	//设置事务传播级别 默认PROPAGATION_REQUIRED 
```
|  spring中事务传播特性   |   |
|  ----  | ----  |
| PROPAGATION_REQUIRED  | 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。 |
| PROPAGATION_REQUIRES_NEW  | 创建一个新的事务，如果当前存在事务，则把当前事务挂起。 |
| PROPAGATION_SUPPORTS  | 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。  |
| PROPAGATION_NOT_SUPPORTED  | 以非事务方式运行，如果当前存在事务，则把当前事务挂起。  |
| PROPAGATION_NEVER  | 以非事务方式运行，如果当前存在事务，则抛出异常  |
| PROPAGATION_MANDATORY  | 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。  |
| PROPAGATION_NESTED  | 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行； 如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。  |


### 事务失效  
原因,除开配置错误，例如忘记配置事务管理器，@EnableTransactionManagement ,事务传播级别设置错误等等
1. 数据库层面：数据库引擎是否支持事务
2. 业务代码层面：
	1. 需要事务管理的类是否被spring管理
	2. @Transactional 注解默认只对public有效
	3. 自调用导致代理逻辑未生效。因为spring中的事务 是通过AOP方式实现，
	本类方法自调用会导致，第一个方法调用第二个方法时，不是通过代理类调用第二个方法的，这就导致了第二个方法的事务未生效
	```Java
	@Service
	public class TestService {
		 @Transactional
		 public void save(A a, B b) {	//调用save()时会通过代理类调用，开启事务
		  	saveB(b);	//在代理类中调用saveB() 则是调用的原本类里的saveB()
		 }

		 @Transactional(propagation = Propagation.REQUIRES_NEW)
		 public void saveB(B b){
		  	dao.saveB(a);
		 }
	}
	```
	


- __事务失效[[参考链接#一个Transaction哪里来这么多坑？ https mp weixin qq com s rz5w2KWzs7ubzO36lGNA4Q|一个Transaction哪里来这么多坑？]] __
- [讲讲 Spring 事务有哪些坑?](https://mp.weixin.qq.com/s/BRRELMbULFL-2eZRSehC7w)