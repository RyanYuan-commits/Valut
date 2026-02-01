---
type: permanent
banner: Assets/Banner/pexels-maxravier-3331094.jpg
---
# 🌐 关键词

数据库, InnoDB, MVCC

---

# 🔖 详细解释

## 1	InnoDB 对于隔离级别的实现

InnoDB 使用 Lock + MVCC 实现不同的隔离级别, 对与**写事务**, 会在**修改记录之前**对这一行记录加锁并持有, 以此来避免冲突的发生; 

对于**只读事务**, 会默认采用多版本并发控制的的方式, 这种方式不需要加锁, 而是在事务第一次读操作 (RR) 或者当前语句开始 (RC) 时, 获取并持有一个当时实例全局的事务活跃状态, 作为自己的 **ReadView**, 相当于对当时的事务状态打了一个 Snapshot, 后续访问某一行的时候, 会根据这一行上面记录的事务 ID, 通过自己持有的 ReadView 来判断是否可见, 如果不可见, 再沿着记录上的 Roll Ptr 去 Undo 中查找自己可见的历史版本. 这种读的方式称之为快照读. 

与之对应的, InnoDB 还支持加锁读(Lock Read)的方式, 当 Select 语句中使用了如 `Select. . . . for Update/Share` 时, 这时查询不再通过 MVCC 的方式, 而是像写操作一样, 先对需要访问的记录加锁, 之后再读取记录内容, 这种方式会跟写请求相互阻塞, 从而读到的也一定是该记录当前最新的值, 因此也被称为当前读.

## 2	ReadView

ReadView 用于确定在事务执行期间哪些记录对其来说是可见的, InnoDB 默认的隔离级别 Read Repeatable, 会在事务的第一次读操作开始时生成一个 ReadView, 而对于加锁读的语句, 则不会生成 Read View, 而是先在尝试在表格上加一个意向锁. Read View 有四个重要的字段:

- `m_ids`:  创建时, 当前数据库中活跃事务的 ids, 
	
- `min_trx_id`: 创建时, 所有活跃事务中最早开启的事务 id, 
	  
- `max_trx_id` : 创建时下一个事务的 id 值, 即当前全局事务中最大的事务 id + 1；
	
- `creator_trx_id` : 创建该 Read View 的事务的事务 id. 

将这些属性画在同一条 X 轴上大致为:

![[ReadView 图例.png|1000]]

事务在执行过程中, 通过 ReadView 中的属性来判断哪些事务在其开始之前就已提交, 这些事务的操作对其是可见的.

```cpp
[[nodiscard]] bool changes_visible(trx_id_t id,   const table_name_t &name) const {  
  // 检查 ID 是否大于零, 如果否会直接返回异常
  ut_ad(id > 0);  

  // 如果 ID 小于等于 min_trx_id , 直接返回 true
  if (id < m_up_limit_id || id == m_creator_trx_id) {  
    return (true);  
  }  

  // 检查 ID 的准确性
  check_trx_id_sanity(id, name);  

  // 如果 ID 大于等于 max_trx_id 直接返回 false
  if (id >= m_low_limit_id) {  
    return (false);  
  } else if (m_ids.empty()) {
  // 如果 ID 在 min_trx_id 到 max_trx_id 之间
    return (true);  
  }  
  // 在 m_jds 中搜索 ID, 如果不存在, 返回 true
  const ids_t::value_type *p = m_ids.data();  
  
  return (!std::binary_search(p, p + m_ids.size(), id));  
}
```

## 3	通过 undo log 寻找历史版本

当读操作发现当前记录自己不可见后, 就需要通过 Undo Log 来寻找历史版本, 通过 Undo Record 记录的 trx_id 结合 ReadView 做可见性判断, 如果不可见就沿着 Record 或 Undo Record 中记录的 rollptr 一路找更老的历史版本. 

![[使用 undo log 查找可见版本.png|1000]]

Undo Log 作为 Logical Log, 记录的其实是前后两个版本的 diff 信息, 而读操作最终是要获得完整的 Record 内容的, 也就是说这个沿着 rollptr 指针一路查找的过程中需要用 Undo Record 中的 diff 内容依次构造出对应的历史版本.