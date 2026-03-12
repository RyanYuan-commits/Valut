---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
---
# 🌐 关键词

Undo Record, InnoDB

---

# 🔖 详细解释

## 1	Insert 类型的 Undo Record

Insert 类型的 undo log 在代码中对应 `TRS_UNDO_INSERT_REC`, 这种 undo log **不需要**支持 MVCC, 所以只需要记录对应 Record 的**主键**, 供回滚时查找 Record 的位置即可.

![[TRX_INDO_INSERT_REC.png|800]]
其中 Undo Number 是 Undo 的一个**递增编号**, Table ID 用来表示是哪张表的修改. 下面一组 Key Fields 的长度不定, 因为对应表的主键可能由多个 field 组成, Undo Record 记录 Record 完整的主键信息, 回滚的时候可以通过这个信息在索引中定位到对应的 Record. 在 Undo Record 的头尾还各留了两个字节记录其前序和后继 Undo Record 的位置.

## 2	Update 类型的 undo record

MVCC 需要保留 Record 的多个历史版本, 当某个 Record 的历史版本还在被使用时(即 Record 的事务 id 大于当前活跃的最小事务 Id 时), 这个 Record 是不能被**真正的删除的**, 针对不同的**操作方式**, MySQL 将 Update 类型的 undo record 划分为以下三种, 它们存储的内容是类似的:

- `TRX_UNDO_UPD_EXIST_REC`: 更新已存在的记录, 是最常见的 Update 操作, 当执行 `UPDATE` 语句修改某一行, 且这一行没有被标记删除;
	
- `TRX_UNDO_DEL_MARK_REC`: 删除标记记录;
	
- `TRX_UNDO_UPD_DEL_REC`: 更新一个已经删除的记录.

![[TRX_UNDO_UPD_EXIST_REC.png|700]]

除了跟 Insert Undo Record 相同的头尾信息, 以及主键之外, Update Undo Record 增加了:

- **Transaction Id**: 记录了产生这个历史版本事务 Id, 用作后续 MVCC 中的版本可见性判断;
	
- **Rollptr**: 指向该记录的上一个版本的位置, 包括 space number, page number 和 page 内的 offset. 沿着 Rollptr 可以找到一个 Record 的所有历史版本.
	
- **Update Fields**: 记录当前这个 Record 版本相对于**其之后的一次**修改的 Delta 信息, 包括所有被修改的 Field 的编号, 长度和历史值.

---

# 📚 参考内容

[MySQL · 引擎特性 · 庖丁解 InnoDB 之 UNDO LOG](http://mysql.taobao.org/monthly/2021/10/01/)