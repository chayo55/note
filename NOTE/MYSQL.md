### 数据库引擎
InnoDB,MyISAM,archive
## 索引
索引(Index)是帮助MySQL高效获取数据的数据结构.
### 聚簇索引、非聚簇索引
![[聚簇索引和非聚簇索引.jpg]]
1. 对于非聚簇索引表来说（右图），表数据和索引是分成两部分存储的，主键索引和二级索引存储上没有任何区别。使用的是B+树作为索引的存储结构，所有的节点都是索引，叶子节点存储的是索引+索引对应的记录的数据。
2. 对于聚簇索引表来说（左图），表数据是和主键一起存储的，主键索引的叶结点存储行数据(包含了主键值)，二级索引的叶结点存储行的主键值。使用的是B+树作为索引的存储结构，非叶子节点都是索引关键字，但非叶子节点中的关键字中不存储对应记录的具体内容或内容地址。叶子节点上的数据是主键与具体记录(数据内容)。
#### 聚簇索引的优点:
1. 当你需要取出一定范围内的数据时，用聚簇索引也比用非聚簇索引好。
2. 当通过聚簇索引查找目标数据时理论上比非聚簇索引要快，因为非聚簇索引定位到对应主键时还要多一次目标记录寻址,即多一次I/O。
3. 使用覆盖索引扫描的查询可以直接使用页节点中的主键值。

#### 聚簇索引的缺点:
1. 插入速度严重依赖于插入顺序，按照主键的顺序插入是最快的方式，否则将会出现页分裂，严重影响性能。因此，对于InnoDB表，我们一般都会定义一个自增的ID列为主键。
2. 更新主键的代价很高，因为将会导致被更新的行移动。因此，对于InnoDB表，我们一般定义主键为不可更新。
3. 二级索引访问需要两次索引查找，第一次找到主键值，第二次根据主键值找到行数据。二级索引的叶节点存储的是主键值，而不是行指针（非聚簇索引存储的是指针或者说是地址），这是为了减少当出现行移动或数据页分裂时二级索引的维护工作，但会让二级索引占用更多的空间。
4. 采用聚簇索引插入新值比采用非聚簇索引插入新值的速度要慢很多，因为插入要保证主键不能重复，判断主键不能重复，采用的方式在不同的索引下面会有很大的性能差距，聚簇索引遍历所有的叶子节点，非聚簇索引也判断所有的叶子节点，但是聚簇索引的叶子节点除了带有主键还有记录值，记录的大小往往比主键要大的多。这样就会导致聚簇索引在判定新记录携带的主键是否重复时进行昂贵的I/O代价。


#### 联合索引、最左前缀匹配

[谈谈索引为什么能提高查询性能？](https://mp.weixin.qq.com/s/DK8zIueMiSsnkz9C77E10Q)

## 事务
![[Transactional#事务特性 ACID]]
![[Transactional#那ACID靠什么保证的呢？]]

### 主从同步
mysql中开启主从同步，需要打开binlog日志文件，在my.cnf文件中配置，
然后在slave主机设置master主机的地址，端口，账号，密码，日志文件，日志起始位置
主从同步的过程：
master主机提交完事务后，记录到binlog`->`
slave连接master，获取binlog`->`
master创建一个线程，将binlog推送到slave`->`
slave开启一个IO线程，读取master推送过来的binlog，写入到中继日志relaylog`->`
slave开启一个sql线程，将relaylog日志中的操作写入slave主机中的数据库，同步完成`->`
slave记录自己的binlog

---
## SQL优化
使用`explain`、`show profiles`，分析sql语句。
```sql
explain select ...
```
![[explain结果.png]]
1. id 
id值越大的语句越先被执行.
2. select_type
	- `SIMPLE`： 表示此查询不包含 UNION 查询或子查询
	- `PRIMARY`： 表示此查询是最外层的查询
	- `SUBQUERY`： 子查询中的第一个 SELECT
	- `UNION`： 表示此查询是 UNION 的第二或随后的查询
	- `DEPENDENT UNION`：UNION 中的第二个或后面的查询语句, 取决于外面的查询
	- `UNION RESULT`, UNION 的结果
	- `DEPENDENT SUBQUERY`: 子查询中的第一个 SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果.
	- `DERIVED`：衍生，表示导出表的SELECT（FROM子句的子查询）
3. table
table表示查询涉及的表或衍生的表.
4. type
	- `system`: 表中只有一条数据， 这个类型是特殊的 const 类型。
	- `const`: 针对主键或唯一索引的等值查询扫描，最多只返回一行数据。const 查询速度非常快， 因为它仅仅读取一次即可。例如：
	`explain select * from user_info where id = 2；`
	- `eq_ref`: 此类型通常出现在多表的 join 查询，表示对于前表的每一个结果，都只能匹配到后表的一行结果。并且查询的比较操作通常是 =，查询效率较高。例如：
	`explain select * from user_info, order_info where user_info.id = order_info.user_id;`
	- `ref`: 此类型通常出现在多表的 join 查询，针对于非唯一或非主键索引，或者是使用了 最左前缀 规则索引的查询。例如：
	`explain select * from user_info, order_info where user_info.id = order_info.user_id AND order_info.user_id = 5;`
	- `range`: 表示使用索引范围查询，通过索引字段范围获取表中部分数据记录。这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中。例如：
	`explain select * from user_info  where id between 2 and 8；`
	- `index`: 表示全索引扫描(full index scan)，和 ALL 类型类似，只不过 ALL 类型是全表扫描，而 index 类型则仅仅扫描所有的索引， 而不扫描数据。index 类型通常出现在：所要查询的数据直接在索引树中就可以获取到, 而不需要扫描数据。
	- `ALL`: 表示全表扫描，这个类型的查询是性能最差的查询之一。如一个查询是 ALL 类型查询， 那么一般来说可以对相应的字段添加索引来避免。

5. possible_keys
它表示 mysql 在查询时，可能使用到的索引。
6. key
此字段是 mysql 在当前查询时所真正使用到的索引。
7. key_len
表示查询优化器使用了索引的字节数，这个字段可以评估组合索引是否完全被使用。
8. ref
这个表示显示索引的哪一列被使用了，如果可能的话,是一个常量。前文的type属性里也有ref，注意区别。
9. rows
rows 也是一个重要的字段，mysql 查询优化器根据统计信息，估算 sql 要查找到结果集需要扫描读取的数据行数，这个值非常直观的显示 sql 效率好坏， 原则上 rows 越少越好。
10. extra   
	- explain 中的很多额外的信息会在 extra 字段显示, 常见的有以下几种内容:   
	- `using filesort`：表示 mysql 需额外的排序操作，不能通过索引顺序达到排序效果。一般有 using filesort都建议优化去掉，因为这样的查询 cpu 资源消耗大。
	- `using index`：覆盖索引扫描，表示查询在索引树中就可查找所需数据，不用扫描表数据文件，往往说明性能不错。
	- `using temporary`：查询有使用临时表, 一般出现于排序， 分组和多表 join 的情况， 查询效率不高，建议优化。
	- `using where` ：表名使用了where过滤。

使用show profiles
```sql
select @@profiling; #查看是否打开profiling功能
set profiling =1;	#打开profiling功能
#sql 语句 select ...
show profiles;
show profile for query 15;

```

[书写高质量SQL的30条建议](https://mp.weixin.qq.com/s/5UPwmWVtnT5WoWTGj7uHLg)
[52 条 SQL 语句性能优化策略](https://mp.weixin.qq.com/s/C4aBqeNzravq7n_SVz3R5g)
1. 不要使用`select *` ，使用`select name,age`具体字段
	- 只查询需要的字段，节省资源，减少网络开销
	- `select *`  有可能不会使用索引，效率大大降低
2. 使用精确的类型，节约存储。
	- varchar和char，char处理速度更快，varchar是可变长度，节省存储。建议varchar
	- ENUM 和 SET，ENUM只能取单值，性别字段适合定义为 ENUM；在需要取多个值的时候，适合使用SET类型，比如：要存储一个人兴趣爱好，最好使用SET类型。ENUM和SET的值是以字符串形式出现的，但在内部，MySQL以数值的形式存储它们。
3. 前导模糊查询会导致索引失效，`like %s`，后置模糊查询不会，`like s%`
4. 数据类型出现隐式转换的时候不会命中索引，特别是当列类型是字符串，一定要将字符常量值用引号引起来。
	```sql
	SELECT id,name FROM user WHERE name=1;
	SELECT id,name FROM user WHERE name='1';
	```
5. 复合索引的情况下，遵从最左前缀原则。注意，最左前缀原则并不是说是查询条件的顺序；而是查询条件中是否包含索引最左列字段。
	```sql
	ALTER TABLE user ADD INDEX index_name (name,age,status);
	
	SELECT id,name FROM user WHERE name='swj' AND status=1;//命中
	SELECT id,name FROM user WHERE status=1 AND name='swj';//命中
	SELECT id,name FROM user WHERE status=2 AND age=22;//未命中，因为where中不包含最左的name
	```
6. 避免在`where`字句中使用`or`连接条件，因为`or`后面的条件如果没有索引，则都不会走索引查询
	```sql
	// 如果id有索引，age没有索引，则会走全表查询
	SELECT id,age,name FROM payment WHERE id = 203 OR age = 33;
	```
7. 负向条件查询不能使用索引，可以优化为`in`查询。负向条件有：`!=`、`<>`、`not in`、`not exists`、`not like`等。
8. 范围条件查询可以命中索引。范围条件有：`<`、`<=`、`>`、`>=`、`between`等。
9. 避免在sql中执行计算，数据库执行计算不会命中索引。
	```sql
	SELECT name,id,age FROM user WHERE age+1>24; //不会使用索引查询
	```
10. 利用覆盖索引进行查询，避免回表。
11. 不适合构建索引的情况
	- **更新十分频繁的字段上不宜建立索引**：因为更新操作会变更B+树，重建索引。这个过程是十分消耗数据库性能的。
	- **区分度不大的字段上不宜建立索引**：类似于性别这种区分度不大的字段，建立索引的意义不大。因为不能有效过滤数据，性能和全表扫描相当。另外返回数据的比例在30%以外的情况下，优化器不会选择使用索引。



--- 
[[参考链接#MySQL夺命连环13问！ https mp weixin qq com s UFb-hqxc_nSfSZBXeOfx8Q|MySQL夺命连环13问！]]

[37 个 MySQL 数据库小技巧](https://mp.weixin.qq.com/s/V6mQeMq9xrV5VXj62ipL_w)

[线上千万级大表排序，如何优化？](https://mp.weixin.qq.com/s/I5lh4ZRDRWlyeDBR7MTAXg)
[https://zhuanlan.zhihu.com/p/61687047](https://zhuanlan.zhihu.com/p/61687047)