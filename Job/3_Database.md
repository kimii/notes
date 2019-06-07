# Database(数据库)
--------
## mysql
Reference: 
- [1] Mysql 技术内幕 InnoDB 存储引擎
- [2] 高性能 MySQL

### Mysql 概述
- 区分数据库和数据库实例
- 单进程多线程的架构
- 存储引擎：
	- InnoDB: 
		1. 支持事务，面向 OLTP(on-line transaction processing，在线事务长处理)
		2. 行锁，支持外键
		3. 使用 MVCC(多版本并发控制)，实现 4 种隔离级别，默认 REPEATABLE 级别，使用 next-key locking 策略避免幻读（？）
	- MyISAM:
		1. 不支持事务，面向 OLAP（on-line anakytical processing，在线分析处理）
		2. 表锁，支全文索引
- 事务
	- 一组原子性的 SQL 查询
	- ACID:
		- A: 原子性 -- 不可分割，要么全部提交成功，要么全部回滚失败
		- C: 一致性 -- 总是从一个一致性状态到另一个一致性状态
		- I: 隔离性 -- 提交之前，对其它事务不可见
		- D: 持久性 -- 提交，所做修改永久保存
	- 隔离级别
		- READ UNCOMMITTED: 未提交读 -- 问题脏读
			- 脏读： 事务可以读取未提交数据
		- READ COMMITTED: 提交读，也叫不可重复读
		- REPEATABLE READ: 可重复读 -- 解决脏读 -- 问题幻读
			- 幻读： 事务读取某个范围的记录，另外事务在该范围插入新行，该事物再次读取产生**幻行**
		- SERIALIZABLE: 可串行化 -- 解决幻读
- MVCC(多版本并发控制)
	- 可认为是**行级锁**的变种，但大多**避免加锁**操作，大都实现非阻塞读，写只锁定必要行
	- 实现： 通过保存数据在某个时间点的快照
		- InnoDB MVCC 简化实现
			- 在每行记录后面保存 2 隐藏列
				1. 行的创建时间
				2. 行的过期时间（或删除时间）
			- 注： 这里时间指系统版本号，每开始一个新的事务，系统版本号会自动递增
			- 解释： 保存额外系统版本号，使得大多数操作都可以不用加锁
	- 分类： 乐观、悲观
	- 与隔离级别： 只能在 REPEATABLE READ 和 READ COMMITTED 下工作
### InnoDB 存储引擎
- 体系架构
	- 后台线程
		1. Master Thread: 将缓冲池的数据异步刷新到磁盘，保证数据一致性
		2. IO Thread: 负责 IO 请求的回调处理，包含 4 种 IO Thread，包括 1 insert buffer thread + 1og thread + 4 read + 4 write thread
		3. Purge Thread: 回收已经使用并分配的 undo 页
		4. Page Cleaner Thread: 将脏页的刷新操作都放入单独的线程中完成
	- 内存
		1. 缓冲池：协调 CPU 速度和磁盘速度之见的鸿沟
			- 简单看是一块内存区域，具体的不仅包括数据页（data page）和索引页（index page），还包括插入缓冲， 自适应 hash 索引，锁信息，数据字典信息
			- 缓冲池大小影响数据库性能
		2. LRU List、Free List 和 Flush List
			- LRU List
				- 基于 LRU 算法的优化
					- midpoint insertion strategy
					- 2 控制参数： innodb_old_block_pct; innodb_old_block_time
				-管理已经读取的页
			- Free List
				- 数据库刚启动，LRU 列表为空，页放入 Free 列表
			- Flush List
				- 脏页列表
				- 注意： 脏页既存在于 LRU 列表中，也存在于 Flush 列表中；但 LRU 列表管理缓冲池中页的可用性，Flush 列表管理将页刷新回磁盘，互不影响
		3. 重做日志缓冲（redo log buffer）
			- 将重做日志信息先放入重做日志缓冲区，再按一定频率将其刷新到重做日志文件
			- 8 M 足够
		4. 额外的内存池
			- 对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请
- checkpoint 技术
	- 解决问题
		- 缩短数据库恢复时间
		- 缓冲池不够用时，将脏页刷新到磁盘
		- 重做日志不可用时，刷新脏页
	- 2 种
		- Sharp Checkpoint: 关闭数据库默认操作，将所有的页够刷新回磁盘，默认方式
		- Fuzzy Checkpoint 
			- 几种情况
				- Master Thread Checkpoint: 以每秒或每 10 秒的速度从缓冲池的脏页列表中刷新一定比例的页回磁盘
				- FLUSH_LRU_LIST Checkpoint: 保证 LRU 列表约 100 个空闲列表可供使用
				- Async/Sync Flush Checkpoint: 重做日志不可用情况（？）
				- Dirty Page too much Checkpoint: 脏页数量太多



### 5 索引与算法

### 6 锁
- 区分 lock 和 latch
	- latch： 闩锁，要求锁定时间短，对象是线程，包括 mutex(互斥量) 和 rwlock(读写锁)
	- lock: 对象是事务，包括行锁、表锁、意向锁

- InnoDB 中锁
	- 行级锁
		- 共享锁： 允许事务读一行数据
		- 排他锁： 允许事务删除或更新一行数据
	- 表锁
		- 意向锁: 为了在一个事务中揭示下一行将被请求的锁类型
			1. 意向共享锁
			2. 意向排他锁

### 7 事务


## redis

## MongoDB