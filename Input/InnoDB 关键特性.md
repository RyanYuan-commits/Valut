---
type: input
banner: Assets/Banner/pexels-tomas-malik-793526-3509971.jpg
draw: "[[Draw Place of InnoDB 存储引擎]]"
---
# 📰 阅读笔记

## 1	插入缓冲

### 1.1	Insert Buffer

在更新非唯一的辅助索引时, 如果索引页不在 Buffer Pool 中, 将更新操作记录在缓冲区专用的 B+ 树中, 之后合并处理. 

- 要求非唯一: 唯一索引更新前, 引擎需要去查询索引页来判断插入记录的唯一性;
	
- 要求辅助索引: 插入不一定是顺序的(与聚簇索引不同), 效率低.

应用程序进行大量插入操作, 但是此时数据库出现了宕机, 势必会有大量的 Insert Buffer 没有合并到实际的非聚簇索引中, 这时的恢复需要很长的时间, 极端情况下甚至需要几个小时.

在写密集的场景下 Insert Buffer 会展用过多的缓冲池内存, 默认最大可以占用到 1/2, 对其他操作会带来一些影响, 可能需要修改.

### 1.2	Change Buffer

1.0.x 版本开始引入, 可看作对 Insert Buffer 的升级, 实现对 INSERT, DELETE, UPDATE 的缓冲, Insert Buffer, Delete Buffer, Purge Buffer.

操作对象仍是非唯一的辅助索引.

### 1.3	Insert Buffer 内部实现

数据结构是一颗 B+ 树, 全局只有一个 Insert Buffer B+ Tree, 存放在**共享表空间**中.

非叶子节点存放的是查询的 search key, 由 space, marker, offset 构成.

- space: 待插入记录所在表的表空间 id, InnoDB 每个表有一个唯一的 space id; 4Byte
	
- marker: 用于兼容老版本的 insert buffer; 1Byte
	
- offset: 页所在的偏移量; 4Byte

当一个非唯一的辅助索引要插入到页 (space, offset) 时, 如果这个页不在 Buffer Pool 中, 那么 InnoDB 会根据上述规则构造一个 serach key, 然后查询 InsertBuffer 这个 B+ 树, 然后将这条记录插入到 insert buffer 的叶子节点中.

叶子节点由 space, marker, offset, metadata, secondary index record 构成:

- space, marker, offset 含义与非叶子节点相同, 占用 9 Bytes;
	
- metadata 占用 4 字节, 其中两个字节存储了每个记录进入 Insert Buffer 的顺序;
	
- 为了保证 Insert Buffer 中的数据最终可以插入索引页, Insert Buffer 中还需要跟踪索引页的状态

### 1.4	Merge Insert Buffer

- 辅助索引页被读取到缓冲池中;
	
- Insert Buffer Bitmap 页追踪到该辅助索引页在插入后可用空间会小于某个值后 (1/32);
	
- Master Thread.

## 2	Double Write 双写

### 2.1	背景

- InnoDB 页大小通常为 16KB, 如果崩溃发生在磁盘内页文件的写入过程中, 就可能会导致页内容损坏, 称为 "部分写";
	
- Redo Log 只能 "重放逻辑/物理修改", 前提是目标页本身是一个合法的旧版本; 所以需要另外一份完整页副本用于修复, 这就是 Double Write 的价值.

### 2.2	架构

![[InnoDB double write 架构.png|800]]

- 内存中的 Double Write Buffer, 大小为 2 MB;
	
- 物理磁盘共享表空间中连续的 128 个页, 即两个区, 2 MB.

刷新 Buffer Pool 脏页时, 先将脏页复制到 Double Wirte Buffer. 然后分两次写到共享表空间的 double write 中.

写入完成后, 再将 Double Write Buffer 中的页写入各个表空间中.

## 3	自适应哈希索引

Adaptive Hash Index, AHI

创建时机: 观察到如果建立哈希索引可以给速度带来提升时;

- 以某个模式访问了 100 次;
	
- 页通过该模式访问了 N 次, N = 页中记录数 / 16.

模式指的是查询条件, 如 WHERE a=xxx; WHERE a=xxx and b=xxx 交替进行, 则不会构建自适应哈希索引.
![[SQL必知必会.epub]]
启动后, 估算辅助索引的连接性能提示 5 倍, 读取写入速度提示 2 倍.

## 4	异步 IO





---

# 💭 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?