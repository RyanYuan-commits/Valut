---
type: input
draw: "[[Draw Place of InnoDB 存储引擎]]"
banner: Assets/Banner/pexels-picjumbo-com-55570-4457409.jpg
---
# 📰 阅读笔记

## 1	InnoDB 的版本

| 版本           | 功能                                       |
| ------------ | ---------------------------------------- |
| 老版本 InnoDB   | 支持 ACID, 行锁设计, MVCC                      |
| InnoDB 1.0.x | 继承了上述版本的所有功能, 增加了 compress 和 dynamic 页格式 |
| InnoDB 1.1.x | 继承了上述版本所有功能, 增加了 Linux AIO, 多回滚段         |
| InnoDB 1.2.x | 继承了上述版本的所有功能, 增加了全文索引支持, 在线索引添加          |

## 2	InnoDB 体系架构

### 2.1	架构总览

![[InnoDB 体系架构.png|700]]

后台线程的主要作用是负责刷新内存池中的数据, 保证缓冲池的内存缓冲是最近的数据, 还负责将已经修改的数据文件刷新会磁盘, 同时保证在数据库发生异常的情况下,  InnoDB 能够恢复到正常状态.

### 2.2	后台线程

- Master Thread: 负责将缓冲池中的数据异步刷新到磁盘, 保证数据的一致性, 包括脏页的刷新, 合并插入缓冲, UNDO 页的回收等;
	
- IO Thread: InnoDB 中大量使用 AIO 来处理 IO 请求, 有四种 write, read, inster buffer 和 log IO thread.
	
- Purge Thread: 事务被提交后, undolog 可能不在需要, 所以需要清除线程来回收已被分配的 undo 页.
	
- Page Cleaner Thread: 在 InnoDB 1.2.x 版本引入的, 目的是将之前版本的脏页刷新操作都放到单独的县城中进行.

### 2.3	内存

#### 2.3.1	缓冲池

基于磁盘的数据库要解决磁盘读取速度和 CPU 运算速度之间的差异, 需要缓冲池;

缓存的内容有: 索引页, 数据页, undo 页, 插入缓冲, 自适应哈希索引, InnoDB 存储的锁信息, 数据字典信息等.

![[InnoDB 内存数据对象.png|900]]

从 InnoDB 1.0.x 开始, 允许有多个缓冲池实例, 每个页根据哈希平均分配到不同的缓冲池中, 加强并发能力.

#### 2.3.2	LRU List, Free List 和 Flush List

InnoDB 使用优化后的 LRU 算法来管理缓冲池, 新读取到的页会放入 midpoint 位置, 这个策略被称为 midpoint insertion strategy, 默认配置下, 这个位置在 LRU 列表长度尾端的 37% 位置.

目的是为了防止某些读取大量数据的操作将热点数据挤出缓冲区, 如索引或数据扫描的操作, 也就是认为前 67% 的是热点数据, 可以根据实际情况调整.

innodb_old_blocks_time 参数用于表示页读取到 mid 位置后要等待多久才会被加入到 LRU 列表的热端.

空闲页使用 Free 列表维护, 可以通过 SHOW ENGINE INNODB STATUS 来观察 LRU 列表和 Free 列表的使用情况和运行状态, 关键字有 page made yong 和 page not made young, 标识页移动到热点区域, 和由于 innodb_old_blocks_time 的限制而没有移动到热点区域.

```
mysql＞SHOW ENGINE INNODB STATUS\G;
***************************1.row***************************
Type:InnoDB
Name:
Status:
=====================================
120725 22:04:25 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 24 seconds
……
Buffer pool size 327679
Free buffers 0
Database pages 307717
Old database pages 113570
Modified db pages 24673
Pending reads 0
Pending writes:LRU 0,flush list 0,single page 0
Pages made young 6448526,not young 0
48.75 youngs/s,0.00 non-youngs/s
Pages read 5354420,created 239625,written 3486063
55.68 reads/s,81.74 creates/s,955.88 writes/s
Buffer pool hit rate 1000/1000,young-making rate 0/1000 not 0/1000
……
```

可以通过 "Buffer pool hit rate 1000/1000,young-making rate 0/1000 not 0/1000" 得出线程池运行状态良好, 命中率达到 100%, 这个值通常不能小于 95%.

InnoDB 从 1.0.x 开始支持压缩页功能, 即将原本 16KB 的页压缩到 1KB, 2KB, 4KB 和 8KB, 由于页大小发生变化, LRU 列表也有了些许变化, 对于非 16KB 的页, 通过 unzip_LRU 列表进行管理的.

```
mysql＞SHOW ENGINE INNODB STATUS\G;
……
Buffer pool hit rate 999/1000,young-making rate 0/1000 not 0/1000
Pages read ahead 0.00/s,evicted without access 0.00/s,Random read ahead 0.00/s
LRU len:1539,unzip_LRU len:156
I/O sum[0]:cur[0],unzip sum[0]:cur[0]
……
```
比如上面的打印中, LRU 列表有 1539 个页, 而 unzip_LRU 列表中有 156 个页, LRU 列表包含了 unzip_LRU 列表中的页.

当 LRU 列表的页被修改后, 称该页为脏页, MySQL 通过 CHECKPOINT 将脏页刷新会磁盘, 脏页在 Flush 列表中维护.

#### 2.3.3	redo log buffer

InnoDB 将 redo log 放入这个缓冲区, 然后按照一定频率将其刷新到磁盘, 这个频率一般为一秒, 所以 redo log buffer 不会设置的很大，默认为 8MB.

redo log 刷新时机:

- Master Thread 每秒执行一次刷新;
	
- 每个事务提交时刷新;
	
- 当 redo log buffer 剩余空间小于 1/2 时执行一次刷新.

#### 2.3.4	额外的内存池

重要但是很容易被忽视的内存池, 用于存放 MySQL 自身的一些数据结构, 如分配了缓冲池, 每个缓冲池还有对应的缓冲控制对象, 这些对象记录了一些如 LRU, 锁, 等待等信息, 这些对象的内存需要从额外的内存池分配, 所以在申请了很大的 InnoDB 缓冲池时, 也需要适当的增加额外内存池的大小.

## 3	Checkpoint 技术

脏页需要被刷新会磁盘, 倘若每次一个页发生变化都将新页刷新回磁盘, 那么这个开销是非常大的, 如果热点数据集中在某几个页中, 数据库的性能将变得非常差; 而如果缓冲池将页的新版本刷新到磁盘时发生了宕机, 那么数据就不能恢复了;

为了避免数据丢失的问题, 当前的事务数据库普遍采用了 Write Ahead Log 策略, 当事务提交时, write ahead redo log, 然后再修改页, 当发生宕机后, 通过 redo log 来完成数据的恢复, 确保事务的 Durability.

Checkpoint 技术要解决如下的问题:

- 缩短数据库的恢复时间;
	
- 缓冲池或 redo log 不够用时, 将脏页刷新到磁盘;

数据库宕机后, 只需要刷新 checkpoint 后的 redo log 就可以了.

InnoDB 通过 Log Sequence Number 来标记版本, LSN 是 8 字节的数字, 在每个页, redo log, Checkpoint 都会记录它们的 LSN.

在 InnoDB 中有两种 Checkpoint, 区别每次刷新时刷新多少脏页:

- Sharp Checkpoint: 将所有的脏页都刷新回磁盘, 默认在数据库关闭时触发, 平时一般不使用, 影响性能;
	
- Fuzzy Checkpoint:
	- Master Thread Checkpoint: 以每秒或每十秒的速度刷新, 过程是异步的, 不会阻塞查询;
		
	- FLUSH_LRU_LIST Checkpoint: 保证 LRU 列表有空闲页可用(差不多 100 个), 不满足就会将 LRU 列表的尾端移除;
		
	- Async/Sync Flush Checkpoint: redo log 不可用的情况下, 需要强制将一些页刷新回磁盘, 保证 redo log 循环使用的可用性.

## 4	Master Thread 工作方式

InnoDB 的主要工作都是在一个单独的后台线程 Master Thread 中完成的.

### 4.1	1.0.x 版本前的 Master Thread

Master Thread 拥有最高的线程优先级, 内部由多个循环组成, 主循环, 后台循环, 刷新循环, 暂停循环, Master Thread 根据数据库的状态从这些循环中切换.

Loop 被称为主循环, 分为每秒钟的操作和每十秒钟的操作, 伪代码:

```java
while(true) {
	for (int i = 0, i < 10; i++) {
		do thing once per seconds
		Thread.sleep(1000);
	}
	do things once per ten seconds
}
```

通过线程休眠来实现, 不是精确的十秒, 在负载大的情况下可能会有延迟.

每秒一次的操作:

- 日志缓冲刷新到磁盘, 即使事务没有提交 (总是)
	
- 合并插入缓冲 (总是)
	
- 至多刷新 100 个 InnoDB 的缓冲池中的脏页到磁盘 (可能)
	
- 如果当前没有用户活动, 则切换到 background loop (可能)

即使某个事务还没有提交, InnoDB 仍然每秒会将 redo log buffer 中的内容刷新到磁盘, 可以解释为什么很大的事务, commit 也是很快的.

合并插入缓冲 (Insert Buffer) 并不是每秒都会发生的. InnoDB存 储引擎会判断当前一秒内发生的IO次数是否小于5次, 如果小于5次, InnoDB认为当前的 IO 压力很小, 可以执行合并插入缓冲的操作.

- 刷新 100 个脏页到磁盘 (可能的情况下);  
	
- 合并至多5个插入缓冲 (总是);  
	
- 将日志缓冲刷新到磁盘 (总是);  
	
- 删除无用的 Undo 页(总是);  
	
- 刷新 100 个或者 10 个脏页到磁盘(总是). 

在以上的过程中, InnoDB 存储引擎会先判断过去 10 秒之内磁盘的 IO 操作是否小于 200 次, 如果是, InnoDB 存储引擎认为当前有足够的磁盘 IO 操作能力, 因此将100个脏页刷新到磁盘.

接着, InnoDB存储引擎会合并插入缓冲. 不同于每秒一次操作时可能发生的合并插入缓冲操作, 这次的合并插入缓冲操作总会在这个阶段进行. 

之后, InnoDB存储引擎会再进行一次将日志缓冲刷新到磁盘的操作. 这和每秒一次时发生的操作是一样的. 

接着 InnoDB 存储引擎会执行 full purge 操作, 即删除无用的 Undo 页. 对表进行 update、delete 这类操作时, 原先的行被标记为删除, 但是因为一致性读的关系, 需要保留这些行版本的信息. 但是在 full purge 过程中, InnoDB 存储引擎会判断当前事务系统中已被删除的行是否可以删除, 比如有时候可能还有查询操作需要读取之前版本的 undo 信息, 如果可以删除, InnoDB 会立即将其删除. InnoDB 存储引擎在执行 full purge 操作时, 每次最多尝试回收 20 个 undo 页. 

然后, InnoDB存储引擎会判断缓冲池中脏页的比例 (buf_get_modified_ratio_pct), 如果有超过70%的脏页, 则刷新100个脏页到磁盘, 如果脏页的比例小于70%, 则只需刷新10%的脏页到磁盘. 

如果当前没用用户活动, 或者数据库关闭, 就会切换到这个循环, background loop 会执行以下的操作:

- 删除无用的 undo 页面;
	
- 合并 20 个插入缓冲;
	
- 跳回到主循环;
	
- 不断刷新 100 个页直到符合条件.

如果 Flush Loop 也没有事情可以做了, InnoDB 会切换到 suspend_loop, 将 Master Thread 挂起, 等待时间的发生.

### 4.2	1.2.x 版本前的 Master Thread

解决上面版本对于 IO 次数硬编码的问题, 比如显示的表明刷新页的数量.

新版本定义了磁盘 IO 的吞吐量, 上面的各个操作实际执行 IO 的次数改为占用这个吞吐量的比例.

### 4.3	1.2.x 版本的 Master Thread

```c
if InnoDB is idle
srv_master_do_idle_tasks(); // 之前每 10s 的操作
else
srv_master_do_active_tasks(); // 之前每 1s 的操作
```

刷新脏页的操作分离到了 Page Cleaner Thread, 减轻了 Master Thread 的工作.

---

# 💭 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?