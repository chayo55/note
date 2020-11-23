## Redis中5个基本数据结构
Redis有5个基本数据结构，`string`、`list`、`hash`、`set`和`zset`。它们是日常开发中使用频率非常高应用最为广泛的数据结构。
### **String 类型**
![[redis_String类型结构.jpg]]
Redis的字符串是**动态字符串**，是可以修改的字符串，内部结构实现上类似于Java的ArrayList，采用**预分配冗余空间**的方式来减少内存的频繁分配，如图中所示，内部为当前字符串实际分配的空间capacity一般要高于实际字符串长度len。当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。需要注意的是字符串最大长度为512M。

| 命令 | 说明 | 案例 |
|---|---|---|
| set | 添加key-value <img width=200/>| set username admin |
|get | 根据key获取数据 | get username|
| strlen|根据key获取值的长度|strlen key|
| exists|	判断key是否存在|	exists name，返回1存在  0不存在 |
|del	|删除redis中的key|	del key|
|Keys	|用于查询符合条件的key	|keys * 查询redis中全部的key <br> keys name 使用占位符获取数据 <br>keys nam* 获取nam开头的数据 |
|mset	|赋值多个key-value|	mset key1 value1 key2 value2 key3 value3 |
|mget |	获取多个key的值|	mget key1 key2|
|append	|对某个key的值进行追加|	append key value|
|type|	检查某个key的类型|	type key|
|select	|切换redis数据库|	select 0-15 redis中共有16个数据库|
|flushdb| 	清空单个数据库|	flushdb|
|flushall|	清空全部数据库	|flushall|
|incr|	自动加1|	incr key|
|decr|	自动减1 |	decr key|
|incrby|	指定数值添加	|incrby 10|
|decrby|	指定数值减|	decrby 10|
|expire|	指定key的生效时间 单位秒	|expire key 20<br>key20秒后失效 |
|pexpire|	指定key的失效时间 单位毫秒|	pexpire key 2000<br>key 2000毫秒后失效|
|ttl	|检查key的剩余存活时间|	ttl key  -2数据不存在  -1该数据永不超时|
|persist	|撤销key的失效时间	|persist key|

---

### **Hash类型**
同java中hashmap一样。
在redis中有扩容，机制同Java一样。
也有缩容，缩小数组为原来大小一半。

|命令|	说明|	案例|
|---|---|---|
|hset	|为对象添加数据	|hset key field value|
|hget|	获取对象的属性值	|hget key field
|hexists	|判断对象的属性是否存在|	HEXISTS key field<br>1表示存在  0表示不存在|
|hdel	|删除hash中的属性	|hdel user field [field ...]|
|hgetall	|获取hash全部元素和值	|HGETALL key|
|hkyes|	获取hash中的所有字段|	       HKEYS key|
|hlen|	获取hash中所有属性的数量|	hlen key|
|hmget|	获取hash里面指定字段的值	|hmget key field [field ...]|
|hmset	|为hash的多个字段设定值|	hmset key field value [field value ...]|
|hsetnx	|设置hash的一个字段,<br>只有当这个字段不存在时有效|	HSETNX key field value|
|hstrlen	|获取hash中指定key的值的长度|	HSTRLEN key field|
|hvals|	获取hash的所有值|	HVALS user|

---

### **List类型**
![[redis_List类型结构.jpg]]
Redis将列表数据结构命名为list而不是array，是因为列表的存储结构用的是**链表**而不是数组，而且链表还是**双向链表**。因为它是链表，所以随机定位性能较弱，首尾插入删除性能较优。如果list的列表长度很长，使用时我们一定要关注链表相关操作的时间复杂度。

快速列表
![[redis_快速列表_ziplist.jpg]]
如果再深入一点，你会发现Redis底层存储的还不是一个简单的linkedlist，而是称之为快速链表quicklist的一个结构。首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也即是压缩列表。它将所有的元素紧挨着一起存储，分配的是一块连续的内存。当数据量比较多的时候才会改成quicklist。因为普通的链表需要的附加指针空间太大，会比较浪费空间。比如这个列表里存的只是int类型的数据，结构上还需要两个额外的指针prev和next。所以Redis将链表和ziplist结合起来组成了quicklist。也就是将多个ziplist使用双向指针串起来使用。这样既满足了快速的插入删除性能，又不会出现太大的空间冗余。

负下标 链表元素的位置使用自然数0,1,2,....n-1表示，还可以使用负数-1,-2,...-n来表示，-1表示「倒数第一」，-2表示「倒数第二」，那么-n就表示第一个元素，对应的下标为0。

队列／堆栈 链表可以从表头和表尾追加和移除元素，结合使用`rpush`、`rpop`、`lpush`、`lpop`四条指令，可以将链表作为队列或堆栈使用，左向右向进行都可以。


|命令|	说明|	案例|
|---|---|---|
|lpush|	从队列的左边入队一个或多个元素	|LPUSH key value [value ...]|
|rpush	|从队列的右边入队一个或多个元素|	RPUSH key value [value ...]|
|lpop	|  从队列的左端出队一个元素|	LPOP key|
|rpop|	从队列的右端出队一个元素	|RPOP key|
|lpushx|	当队列存在时从队列的左侧入队一个元素	|LPUSHX key value|
|rpushx |	当队列存在时从队列的右侧入队一个元素|	RPUSHx key value|
|lrange|	从列表中获取指定返回的元素|	  LRANGE key start stop<br>Lrange key 0 -1 获取全部队列的数据|
|lrem|	从存于 key 的列表里移除前 count 次出现的值为 value 的元素。这个 count 参数通过下面几种方式影响这个操作：<br>count > 0: 从头往尾移除值为 value 的元素。<br>count < 0: 从尾往头移除值为 value 的元素。<br>count = 0: 移除所有值为 value 的元素。| LREM list -2 “hello”<br> 会从存于 list 的列表里移除最后两个出现的 “hello”。<br>需要注意的是，如果list里没有存在key就会被当作空list处理，<br>所以当 key 不存在的时候，这个命令会返回 0。|
|Lset| 	设置 index 位置的list元素的值为 value|	LSET key index value|

---

### set
同Java中hashSet一样，内部为hashmap，value都指向同一个内部值。

---

### sortedSet
SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构`Map<String, Double>`，可以给每一个元素value赋予一个权重`score`，另一方面它又类似于`TreeSet`，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。
zset底层实现使用了两个数据结构，第一个是hash，第二个是跳跃列表，hash的作用就是关联元素`value`和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。跳跃列表的目的在于给元素value排序，根据score的范围获取元素列表。

>跳跃列表 zset内部的排序功能是通过「跳跃列表」数据结构来实现的，它的结构非常特殊，也比较复杂。这一块的内容深度读者要有心理准备。
因为zset要支持随机的插入和删除，所以它不好使用数组来表示。我们先看一个普通的链表结构。
![[redis_跳跃列表图1.jpg]]
我们需要这个链表按照score值进行排序。这意味着当有新元素需要插入时，需要定位到特定位置的插入点，这样才可以继续保证链表是有序的。通常我们会通过二分查找来找到插入点，但是二分查找的对象必须是数组，只有数组才可以支持快速位置定位，链表做不到，那该怎么办？
想想一个创业公司，刚开始只有几个人，团队成员之间人人平等，都是联合创始人。随着公司的成长，人数渐渐变多，团队沟通成本随之增加。这时候就会引入组长制，对团队进行划分。每个团队会有一个组长。开会的时候分团队进行，多个组长之间还会有自己的会议安排。公司规模进一步扩展，需要再增加一个层级——部门，每个部门会从组长列表中推选出一个代表来作为部长。部长们之间还会有自己的高层会议安排。
跳跃列表就是类似于这种层级制，最下面一层所有的元素都会串起来。然后每隔几个元素挑选出一个代表来，再将这几个代表使用另外一级指针串起来。然后在这些代表里再挑出二级代表，再串起来。最终就形成了金字塔结构。
想想你老家在世界地图中的位置：亚洲-->中国->安徽省->安庆市->枞阳县->汤沟镇->田间村->xxxx号，也是这样一个类似的结构。
![[redis_跳跃列表图2.jpg]]
「跳跃列表」之所以「跳跃」，是因为内部的元素可能「身兼数职」，比如上图中间的这个元素，同时处于L0、L1和L2层，可以快速在不同层次之间进行「跳跃」。
定位插入点时，先在顶层进行定位，然后下潜到下一级定位，一直下潜到最底层找到合适的位置，将新元素插进去。你也许会问那新插入的元素如何才有机会「身兼数职」呢？
跳跃列表采取一个随机策略来决定新元素可以兼职到第几层，首先L0层肯定是100%了，L1层只有50%的概率，L2层只有25%的概率，L3层只有12.5%的概率，一直随机到最顶层L31层。绝大多数元素都过不了几层，只有极少数元素可以深入到顶层。列表中的元素越多，能够深入的层次就越深，能进入到顶层的概率就会越大。
这还挺公平的，能不能进入中央不是靠拼爹，而是看运气。


---

## Redis持久化策略
两种持久化策略 RDB()和AOF(appendonly)
1. RDB模式：Redis的默认策略，记录的是**内存数据的快照**，RDB模式是定期持久化，最新的快照会覆盖之前的，所以可能会有数据丢失。<br>RDB模式的配置:  
	- 配置持久化文件名
	- 配置持久化文件路径
	- 配置持久化策略 
		```
		save 900 1		//900秒内 1次更新 持久化一次
		save 300 10		//300秒内 10次更新 持久化一次
		save 60 10000	//60秒内 10000次更新 持久化一次
		```
2. AOF模式：
	- AOF模式默认条件下是关闭的,需要用户手动的开启`appendonly yes`
	- AOF模式是异步的操作 记录的是用户的操作的过程 可以防止用户的数据丢失
	- 由于AOF模式记录的是程序的运行状态 所以持久化文件相对较大,恢复数据的时间长.需要人为的优化持久化文件
		```
		appendfsync always		//操作一次持久化一次
		appendfsync everysec	//每秒持久化一次
		appendfsync no
		```
3. 两种模式的选择：
	1. 如果不允许数据丢失 使用AOF方式
	2. 如果追求效率 运行少量数据丢失 采用RDB模式
	3. 如果既要保证效率 又要保证数据 则应该配置redis的集群 主机使用RDB 从机使用AOF

## Redis内存策略
1. 内存优化算法
	- **LRU算法**：Least Recently Used，最近最少使用。最好用的内存优化算法.
	该算法赋予每个页面一个访问字段，用来记录一个页面自上次被访问以来所经历的时间 t，当须淘汰一个页面时，选择现有页面中其 t 值最大的，即最近最少使用的页面予以淘汰。
	- **LFU算法**：（least frequently used (LFU) page-replacement algorithm）。即最不经常使用页置换算法。要求在页置换时置换引用计数最小的页，因为经常使用的页应该有一个较大的引用次数。但是有些页在开始时使用次数很多，但以后就不再使用，这类页将会长时间留在内存中，因此可以将引用计数寄存器定时右移一位，形成指数衰减的平均使用次数。
	- **RANDOM算法**：随机淘汰。
	- **TTL算法**：把设定了超时时间的数据将要移除的提前删除的算法.
2. redis中内存优化策略
	- volatile-lru 设定了超时时间的数据采用lru算法
	- allkeys-lru 所有的数据采用LRU算法
	- volatile-lfu 设定了超时时间的数据采用lfu算法删除
	- allkeys-lfu 所有数据采用lfu算法删除
	- volatile-random 设定超时时间的数据采用随机算法
	- allkeys-random 所有数据的随机算法
	- volatile-ttl 设定超时时间的数据的TTL算法
	- noeviction 如果内存溢出了 则报错返回. 不做任何操作. 默认值
`maxmemory-policy volatile-lru`
3. 缓存穿透
请求访问没有的数据，因为数据库中没有该数据，所以缓存中也没有，因此请求直接访问数据库，这种情况称为缓存穿透。
**导致问题**：高并发情况下，黑客攻击下，大量访问请求，造成数据库压力过大。
**解决方案：**
	- 网关层面，限制ip，限制用户等访问。
	- 业务层面，缓存空数据，但在请求量大的情况下，会缓存过多空数据，内存占用，![[缓存流程.jpg]]
4. 缓存击穿
大量请求访问同一个key，恰好此时因为某种原因(过期、删除等)，这个key失效了，redis中没有数据，但数据库中有，大量请求直接访问数据库的情况，称为 缓存击穿。
**解决方案：**
	- 不要设定相同的失效时间
	- redis集群
	-  多级缓存
5. 缓存雪崩
高并发情况下，redis中大量数据失效，造成大量请求直接访问数据库导致崩溃，这种情况称为缓存雪崩。
**解决方案：**
	- 不要设定相同的失效时间
	- redis集群
	-  多级缓存
## Redis分片和哨兵机制
如果需要缓存海量数据，单台Redis效率就太低了，即使有足够的内存，也会浪费时间在寻址上，所以需要采用分片机制，启用多台Redis实例。
在Java中将多台redis当做一个整体，采用**一致性hash算法**，将数据存储到不同的Redis实例。
```Java
List<JedisShardInfo> shardInfos = new ArrayList<>();
shardInfos.add(new JedisShardInfo("192.168.126.129", 6379));
shardInfos.add(new JedisShardInfo("192.168.126.129", 6380));
shardInfos.add(new JedisShardInfo("192.168.126.129", 6381));
//...

ShardedJedis shardedJedis = new ShardedJedis(shardInfos);
shardedJedis.set("shards", "redis shards test");
System.out.println(shardedJedis.get("shards"));
```
哨兵机制：为了保证Redis的高可用，配置Redis主从实例。
```shell
// 在slave实例中设置master实例
// slaveof host port
slaveof 192.168.126.129 6379
```
通过心跳机制，哨兵检查Redis主机状态，三次未响应则选择一台slave提升为master，原master修复上线后，改为slave
哨兵配置: `sentinel.conf`配置文件
- 修改保护模式
`protected-mode no`
- 开启后台运行
`daemonize yes`
- 设定哨兵监控
```shell
// 哨兵名称 mymaster 
// 1 表示哨兵集群中投票(是否更换主机)生效的票数 3台哨兵则2票生效 1台哨兵则1票生效
sentinel monitor mymaster 192.168.126.129 6379 1
```
- 修改宕机的时间
```shell
// 主机宕机多长时间更换主机
// Default is 30 seconds.
sentinel down-after-milliseconds mymaster 30000
```
- 选举失败的时间
```shell
// 故障转移 Default is 3 minutes.
// 如果选举投票更换主机，新主机在该时间内未上线，则重新选举
sentinel failover-timeout mymaster 180000

```
- 启动哨兵服务
`redis-sentinel sentinel.conf`
## Redis集群
Redis分片实现了数据的扩容
Redis哨兵实现了数据的高可用
同时实现扩容，和高可用则需要Redis集群
### 配置Redis集群：
在每台Redis实例的`redis.conf`中配置:
- 注释本地绑定IP地址
	```shell
	#bind 127.0.0.1
	```
- 关闭保护模式
	```shell
	protected-mode no
	```
- 修改端口号
	```shell
	port 7000
	```
- 启动后台启动
	```shell
	daemonize yes
	```
- 修改pid文件
	```shell
	pidfile /usr/local/src/redis/cluster/7000/redis.pid
	```
- 修改持久化文件路径
	```shell
	dir /usr/local/src/redis/cluster/7000/
	```
- 设定内存优化策略
	```shell
	maxmemory-policy volatile-lru
	```
- 关闭AOF模式
	```shell
	appendonly no
	```
- 开启集群配置
	```shell
	cluster-enabled yes
	```
- 开启集群配置文件
	```shell
	cluster-config-file nodes.conf
	```
- 修改集群超时时间
	```shell
	cluster-node-timeout 15000
	```

```shell
// --cluster-replicas 1的1这里的值，表示
redis-cli --cluster create --cluster-replicas 1 192.168.126.129:7000 192.168.126.129:7001 192.168.126.129:7002 192.168.126.129:7003 192.168.126.129:7004 192.168.126.129:7005

```
### Redis集群中选举机制
- 选举机制
当集群中一台**主机宕机**，由**剩下的主机投票**选举**宕机主机的从机**作为新主机。如果出现平票，则再次投票，如果连续三次平票，该情况称之为，选举机制中的**脑裂现象**
通过增加主节点以降低脑裂发生的机率。
- 集群崩溃
当主机没有从机时，借用其他主机的从机。当主机没有从机且没有可借用的从机时，集群崩溃。
- Redis集群中的hash槽
Redis集群中有16384个slot(槽)，一台主机占用一个slot，所以Redis集群中最多配置16384台主机。所有的key使用哈希函数(CRC16[key]%16384)分配到不同的slot中。

---

## Redis的建议规范
#### 一、键值设计
1. key名设计  
	- 可读性、可管理性  
	以业务名(或数据库名)为前缀(防止key冲突),业务名:表名:id  
	`user:id:1`
	- 简洁性
	保证语义前提下，控制key的长度，因为内存占用
	`user:{uid}:friends:messages:{mid}` 可简化为 `u:{uid}:fr:m:{mid}`  
	- 不包含特殊字符
	空格、换行符、单双引号、及其他转义字符  

2. value设计
	- 拒绝bigkey 
	string类型控制在10KB以内，hash、list、set、zset元素个数不要超过5000。防止查询慢。
	对已有的bigkey，非string的，不要使用del直接删除，避免造成阻塞(同时注意bigkey的自动过期时间出发的del操作)。  
	- 选择合适的数据类型
		```
		反例：
		set user:1:name tom
		set user:1:age 19
		set user:1:favor football

		正例：
		set user:1 name tom age 19 favor football
		```
	- 控制key的生命周期


#### 二、命令使用
1. 禁用命令
为了安全考虑，禁止线上使用keys、flushall、flushdb等，通过redis的rename机制禁掉命令，或者使用scan的方式渐进式处理。
2. 合理使用select
redis的多数据库较弱，使用数字进行区分，很多客户端支持较差，同时多业务用多数据库实际还是单线程处理，会有干扰。
3. 使用批量操作提高效率
原生命令：例如mget、mset。
非原生命令：可以使用pipeline提高效率。
但要注意控制一次批量操作的元素个数(例如500以内，实际也和元素字节数有关)。
注意两者不同：
原生是原子操作，pipeline是非原子操作。
pipeline可以打包不同的命令，原生做不到
4. 不建议过多使用Redis事务功能
5. Redis集群版本在使用Lua上有特殊要求

#### 三、客户端使用
1. 避免多个应用使用一个Redis实例
不相干的业务拆分，公共数据做服务化。
2. 使用连接池
可以有效控制连接，同时提高效率，标准使用方式：
```Java
Jedis jedis = null;
try {
	jedis = jedisPool.getResource();
	jedis.executeCommand();
} catch (Exception e) {
	logger.error("op key {} error: " + e.getMessage(), key, e);
} finally {
	if (jedis != null)
		jedis.close();
}
```
3. 熔断功能
高并发下建议客户端添加熔断功能(例如netflix hystrix)
4. 合理的加密
设置合理的密码，如有必要可以使用SSL加密访问（阿里云Redis支持）
5. 淘汰策略
根据自身业务类型，选好maxmemory-policy(最大内存淘汰策略)，设置好过期时间。
默认策略是volatile-lru，即超过最大内存后，在过期键中使用lru算法进行key的剔除，保证不过期数据不被删除，但是可能会出现OOM问题。

其他策略如下： | 
------------ | ------------
allkeys-lru | 根据LRU算法删除键，不管数据有没有设置超时属性，直到腾出足够空间为止。
allkeys-random | 随机删除所有键，直到腾出足够空间为止。
volatile-random | 随机删除过期键，直到腾出足够空间为止。
volatile-ttl | 根据键值对象的ttl属性，删除最近将要过期数据。如果没有，回退到noeviction策略。
noeviction | 不会剔除任何数据，拒绝所有写入操作并返回客户端错误信息"(error) OOM

command not allowed when used memory"，此时Redis只响应读操作。
#### 四、相关工具
1. 数据同步
redis间数据同步可以使用：redis-port
2. big key搜索
redis大key搜索工具
3. 热点key寻找
内部实现使用monitor，所以建议短时间使用facebook的redis-faina 阿里云Redis已经在内核层面解决热点key问题

[[参考链接#阿里推荐的Redis使用规范，Redis就要这么用 https my oschina net u 3077716 blog 4401828|阿里推荐的Redis使用规范，Redis就要这么用]]

---

### [[分布式锁]]在Redis的实现
https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/

[Redis常见对象类型的底层数据结构](https://mp.weixin.qq.com/s/qtWKQRnqei09frrIerGvvg)