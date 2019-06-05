# Database(数据库)
--------
## mysql
Reference: Mysql 技术内幕 InnoDB 存储引擎
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
		2. 表锁，支持全文索引
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

### 7 事务


## redis

## MongoDB