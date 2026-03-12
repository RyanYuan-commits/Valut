---
type: permanent
---
# 🌐 核心观点

理清 Java 时间 API 的过去, 现在和底层逻辑.

---

# 🔖 详细解释

## 1	java.util.Date 与 Calendar

在 Java 8 之前, 处理时间主要靠 java.util.Date 和后来的 java.util.Calendar.

### 1.1	java.util.Date

这个类设计得非常糟糕:

- 年份偏移: 它的年份是从 1900 年开始算的(比如要表示 2024 年, 你得传 124).
	
- 月份反人类: 月份从 0 开始(1 月是 0).
	
- 可变性 (Mutable): 这是最致命的. 可以随时通过 `setTime()` 修改它的值. 在多线程环境下, 它是线程不安全的.

### 1.2	Calendar

为了解决 Date 的**国际化问题**, 官方推出了 Calendar. 

虽然解决了时区问题, 但代码变得极其臃肿, 且依然没有解决**可变性和线程安全问题**.

## 2	JSR-310 与 java.time

受开源库 **Joda-Time** 的启发, Java 8 引入了全新的 `java.time` 包. 这套 API 的设计理念彻底颠覆了以往.

核心设计原则:

- 不可变性 (Immutability): 所有类都是 **final** 的, 修改操作会返回一个新对象.
	
- 职责清晰: 区分了 "机器时间" 和 "人类时间".
	
- 遵循 ISO 标准: 默认采用 ISO-8601 日期系统.


---

# 📚 参考内容

