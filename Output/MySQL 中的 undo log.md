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



## 4	undo record 的组织方式

### 4.1	逻辑组织方式 - undo log

每个事务会修改一组 Record, 对应的会产生一组 undo record, 这些 undo record 首尾相连就组成了这个事务的 undo log, 除了一个个 undo record 以外, 还有一个 Undo Record Header 来记录一些必要的控制信息, 一个 undo log 的结构如下:

![[Undo Log.png|800]]



---

# 📚 参考内容

[MySQL · 引擎特性 · 庖丁解 InnoDB 之 UNDO LOG](http://mysql.taobao.org/monthly/2021/10/01/)