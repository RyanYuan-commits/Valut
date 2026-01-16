---
type: permanent
---
# 🌐 核心观点

这些类描述的是人类眼中的日期和时间, 类似于挂在某个房间的墙上的日历和时钟.

---

# 🔖 详细解释

## 1	核心定义: 什么是 Local?

在 java. time 中, "Local" 指的是 "墙上时间" (Wall Clock Time), 有以下特点:

- **脱离坐标**: 它描述的是日历上的日期或时钟上的时刻, 但**不包含时区信息**.
	
- **物理意义**: 它不代表宇宙时间轴上的绝对点. `2026-05-20 13:14` 在没有时区的情况下, 只是一个"描述", 无法确定它究竟发生在什么时候.

## 2	家族成员与创建方法

### 2.1	LocalDate

仅包含日期, 年-月-日.

```java
LocalDate date = LocalDate.now();
LocalDate date = LocalDate.of(2026, 1, 13);
```

### 2.2	LocalTime

仅包含时间, 时:分:秒.

```java
LocalTime time = LocalTime.now();
LocalTime time = LocalTime.of(14, 30, 0);
```

### 2.3	LocalDateTime

表示日期 + 时间.

```java
LocalDateTime dateTime = LocalDateTime.now();
LocalDateTime dateTime = LocalDateTime.of(date, time);
```

推荐使用 `parse()` 方法从符合 ISO-8601 标准的字符串中解析, 如: `LocalDate.parse("2026-01-13")`

## 3	常用 API 操作

由于不可变性, 所有的修改方法都会返回新的实例.

- **"取" (Get)**: `getYear()`, `getMonthValue()`, `getDayOfWeek()`, `getHour()` 等.
	
- **"改" (With)**: 直接修改某个字段.
	
	- `date.withYear(2025)`: 返回同年份改为 2025 的新对象.
		
	- `date.with(TemporalAdjusters.lastDayOfMonth()):` 调整到本月最后一天.
	
- **"加减" (Plus/Minus)**: `date.plusDays(5)`, `time.minusHours(2)`.
	
- **"比较" (Compare)**: `isBefore()`, `isAfter()`, `equals()`.

## 4	核心转换

### 4.1	内部转换

```java
LocalDateTime dt = LocalDateTime.now();

// 1. 拆分
LocalDate date = dt.toLocalDate();
LocalTime time = dt.toLocalTime();

// 2. 合并
LocalDateTime newDt = date.atTime(time); 
LocalDateTime newDt2 = time.atDate(date);
```

### 4.2	升级到绝对时间

```java
LocalDateTime ldt = LocalDateTime.now();

// 1. 赋予时区, 变为 ZonedDateTime
ZonedDateTime zdt = ldt.atZone(ZoneId.of("Asia/Shanghai"));

// 2. 转换为 Instant (需要明确偏移量)
Instant instant = ldt.toInstant(ZoneOffset.ofHours(8));
```

---

# 📚 参考内容

