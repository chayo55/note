### 数据库引擎
innodb
### 索引
索引(Index)是帮助MySQL高效获取数据的数据结构.
#### 索引类型
#### 聚簇索引、覆盖索引
#### 联合索引、最左前缀匹配

### 事务
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
### SQL优化
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
1. 不要使用`select *` ，使用`select name,age`具体字段
	- 只查询需要的字段，节省资源，减少网络开销
	- `select *`  有可能不会使用索引，效率大大降低

2. 如果知道查询结果只有一条，建议用`limit 1`
	- eg:`select id,name from user where name='john' limit 1`
	- 使用`limit`，在找到对应记录后，就不会继续向后扫描了，大大提高效率
	- 但如果`name`字段是唯一索引，就不必了

3. 避免在`where`字句中使用`or`连接条件
	- 


--- 
[[参考链接#MySQL夺命连环13问！ https mp weixin qq com s UFb-hqxc_nSfSZBXeOfx8Q|MySQL夺命连环13问！]]

[37 个 MySQL 数据库小技巧](https://mp.weixin.qq.com/s/V6mQeMq9xrV5VXj62ipL_w)