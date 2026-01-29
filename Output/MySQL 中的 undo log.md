---
type: permanent
banner: Assets/Banner/pexels-faikackmerd-1025469.jpg
---
# 🌐 关键词

WAL, undo log, MySQL

---

# 🔖 详细解释

## 1	为什么需要 undo log

**支持事务回滚**: 在事务触发回滚时, MySQL 需要知道事务中进行了哪些操作;

**支持多版本并发控制**: 在不同的事务隔离级别中, 每个事务看到的表视图是不同的, MySQL 需要知道哪个事务应该看到记录的哪个版本. 

## 2	undo log 与表的连接方式

在 InnoDB 存储引擎中, 每一行数据除了用户自定义的列以外, 还存在几个隐藏列, 其中与 undo log 相关的有 `DB_TRX_ID` 和 `DB_ROLL_PTR`:

- `DB_TRX_ID` 记录最近一次修改这行数据的事务 Id;
	
- `DB_ROLL_PTR` 连接到一条 undo log 记录, 多个 undo log 记录之间相连, 形成一个版本链.

当事务修改了表中的一行数据时, MySQL 会将修改前的旧值拷贝一份, 写入到 undo log 的缓冲区中; 然后 MySQL 更新用户列和隐藏列的内容, 将 `DB_ROLL_PTR` 与生成的 Undo Log 相连.

## 3	undo record 中的内容

InnoDB 修改 Record 时, 需要将历史版本写入一个 undo log 中, 这种 undo log 是 Update 类型的; 而对于插入记录, 还没有历史版本时, 也会写入一个 Insert 类型的 undo log 用以支持回滚时逆向的删除操作.

### 3.1	Insert 类型的 undo record

Insert 类型的 undo log 在代码中对应 `TRS_UNDO_INSERT_REC`, 这种 undo log 不需要支持 MVCC, 所以只需要记录对应 Record 的主键, 供回滚时查找 Record 的位置即可.

![[TRX_INDO_INSERT_REC.png|800]]

- 其中 Undo Number 是 Undo 的一个递增编号, Table ID 用来表示是哪张表的修改. 
	
- 下面一组 Key Fields 的长度不定, 因为对应表的主键可能由多个 field 组成, 这里需要记录 Record 完整的主键信息, 回滚的时候可以通过这个信息在索引中定位到对应的 Record.
	
- 除此之外, 在 Undo Record 的头尾还各留了两个字节用户记录其前序和后继 Undo Record 的位置.

### 3.2	Update 类型的 undo record

MVCC 需要保留 Record 的多个历史版本, 当某个 Record 的历史版本还在被使用时, 这个 Record 是不能被**真正的删除的**.当需要删除时, 其实只是修改对应 Record 的 Delete Mark 标记. 对应的, 如果这时这个 Record 又重新插入, 其实也只是修改一下 Delete Mark 标记, 也就是将这两种情况的 delete 和 insert 转变成了 update 操作. 再加上常规的 Record 修改, 因此这里的 Update Undo Record 会对应三种 Type: TRX_UNDO_UPD_EXIST_REC, TRX_UNDO_DEL_MARK_REC 和 TRX_UNDO_UPD_DEL_REC. 他们的存储内容也类似:

---

# 📚 参考内容

[MySQL · 引擎特性 · 庖丁解 InnoDB 之 UNDO LOG](http://mysql.taobao.org/monthly/2021/10/01/)