---
source:
type: input
---
# 📰 阅读笔记

## 1	为什么需要 Redo Log？

Buffer Pool 提高了读写效率, 但它是基于内存的, 如果发生断电重启, 还没来得及落盘的脏页数据就会丢失. 为了防止数据丢失, 当有一条记录需要更新的时候, InnoDB 引擎会执行这两步操作：

1. 更新 Buffer Pool, 同时标记为脏页;
	
2. 将本次对这个页的修改以 Redo Log 的形式记录下来. 

Redo Log 是物理日志, 记录了某个数据页做了什么修改, 比如对 XXX 表空间中的 YYY 数据页 ZZZ 偏移量的地方做了 AAA 更新, 每当执行一个事务就会产生这样的一条或者多条物理日志, 在事务提交时, 会先将 Redo Log 持久化到磁盘. 当系统崩溃时, 虽然 Buffer Pool 中的脏页数据没有被持久化, 但是 Redo Log 已经持久化, 接着 MySQL 重启后, 可以根据 Redo Log 的内容, 将所有数据恢复到最新的状态. 

## 2	写入原理

### 2.1	顺序写与随机写

顺序写是把数据按相邻逻辑块依次写入磁盘, 位置连续, 几乎不需要额外定位操作; 随机写是把数据写到相距较远的逻辑块, 位置不连续, 会频繁发生寻道.

在传统机械硬盘上, 顺序写通常比随机写快一个甚至多个数量级, 在 SSD 上顺序写页更有利于性能和寿命.

Redo Log 采用的是顺序写方式, 写入开销较小, 需要 Redo Log 的原因:

- 实现事务的持久性, 让 MySQL 有 crash-safe 的能力, 能够保证 MySQL 在任何时间段突然崩溃, 重启后之前**已提交**的记录都不会丢失；
	
- 将写操作从「随机写」变成了「顺序写」, 提升 MySQL 写入磁盘的性能. 

### 2.2	缓冲机制

产生的 Redo Log 不是直接写入磁盘的, 而是在内存中设有一个 Redo Log Buffer, 每当产生一条 Redo Log 时, 会先写入到缓存区, 后续再持久化到磁盘：

![[Redo Log Buffer.png|600]]
 
Redo Log buffer 默认大小 16 MB, 可以通过 innodb_log_Buffer_size 参数动态的调整大小, 增大它的大小可以让 MySQL 处理大事务是不必写入磁盘, 进而提升写 IO 性能. 

Redo Log buffer 数据的刷盘操作会在下列场景中触发: 

- MySQL 正常关闭时;
	
- 当 Redo Log buffer 中记录的写入量大于 Redo Log buffer 内存空间的一半时, 会触发落盘;
	
- InnoDB 的后台线程每隔 1 秒, 将 Redo Log buffer 持久化到磁盘;
	
- 每次事务提交时都将缓存在 Redo Log buffer 里的 Redo Log 直接持久化到磁盘. 

### 2.3	循环写

默认情况下, InnoDB 设有一个 Redo Log 文件组, 由有两个名为 ib_logfile0 和 ib_logfile1 的 Redo Log 文件构成. 在重做日志组中, 每个日志文件的大小是固定且一致的, 假设每个日志文件设置的上限是 1 GB, 那么总共就可以记录 2GB 的操作. 

重做日志文件组是以**循环写**的方式工作的, 从头开始写, 写到末尾就又回到开头, 相当于一个环形. InnoDB 存储引擎会先写 ib_logfile0 文件, 当 ib_logfile0 文件被写满的时候, 会切换至 ib_logfile1 文件, 当 ib_logfile1 文件也被写满时, 会切换回 ib_logfile0 文件. 

Redo Log 是为了防止 Buffer Pool 中的脏页丢失而设计的, 那么如果随着系统运行, Buffer Pool 的脏页刷新到了磁盘中, 那么 Redo Log 对应的记录也就没用了, 这时候就可以擦除这些旧记录, 以腾出空间记录新的日志. 

InnoDB 使用 write_pos 表示 Redo Log 当前记录写到的位置, 用 checkpoint 表示当前要擦除的位置: 

![[Redo Log 循环写.png|500]]

如果 write pos 追上了 checkpoint, 就意味着Redo Log 文件满了, 这时 MySQL 不能再执行新的更新操作, 也就是说 MySQL 会被阻塞, 直到将 Buffer Pool 中的脏页刷新到磁盘中, 然后标记 Redo Log 哪些记录可以被擦除, 接着对旧的 Redo Log 记录进行擦除, 等擦除完旧记录腾出了空间, checkpoint 就会往后移动. 


---

# 💭 我的思考

在 "部分写" 的场景下, 会导致页面本身收到损坏, 此时仅靠 Redo Log 是无法进行恢复的, 用户需要一个页的副本, 此时就需要 double write 技术.