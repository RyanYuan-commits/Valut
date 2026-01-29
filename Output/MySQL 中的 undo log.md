---
type: permanent
banner: Assets/Banner/pexels-faikackmerd-1025469.jpg
---
# 🌐 关键词

WAL, undo log, MySQL

---

# 🔖 详细解释

## 1	为什么需要 undo log

- **支持事务回滚**: 在事务触发回滚时, MySQL 需要知道事务中进行了哪些操作;
	
- **支持多版本并发控制**: 在不同的事务隔离级别中, 每个事务看到的表视图是不同的, MySQL 需要知道哪个事务应该看到记录的哪个版本. 

## 2	undo log 的结构




---

# 📚 参考内容

