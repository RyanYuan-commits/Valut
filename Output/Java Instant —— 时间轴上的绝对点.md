---
type: permanent
---
# 🌐 核心观点

Instant 表示时间轴上的一个瞬时点, 是绝对的机器时间. 它不关心地理位置, 只关心“宇宙中这一刻”的进度.

---

# 🔖 详细解释

## 1	底层原理

**基准线**:以 Unix 纪元 (1970-01-01T00:00:00Z UTC) 为基准点 (Epoch).

**高精度**:内部通过两个字段存储:

- `private final long seconds;`(自纪元以来的秒数)
	
- `private final int nanos;` (当前秒内的纳秒数, 0 到 999, 999, 999).

**不含时区**: 它始终是 UTC 时间. 输出格式通常以 Z 结尾 (如 2026-01-12T16:54:40. 123Z).

## 2	常用 API

### 2.1	工厂方法

```java
// 获取当前的 UTC 时刻
Instant now = Instant.now();

// 从 Epoch 创建
Instant instant = Instant.ofEpochMilli(epochMilli);
Instant instant = Instant.ofEpochSecond(epochMilli / 1000);
```

### 2.2	比较与转换方法

```java
// 返回 epoch 秒
long seconds = instant.getEpochSecond(); 

// 返回纳秒
int nanos = instant.getNano();

// 转 ZonedDateTime
ZonedDateTime zdt = instant.atZone(ZoneId.of("Asia/Hong_Kong"));

// 字符串表示
String str = instant.toString();

// 比较 
int cmp = instant.compareTo(Instant.EPOCH);

// 是否在前 / 后
boolean isAfter = instant.isAfter(Instant.EPOCH);
boolean isBefore = Instant.EPOCH.isBefore(instant);
```

### 2.3	调整与加减方法

```java
// 加减 Duration
Instant plusDur = instant.plus(Duration.ofSeconds(10));
Instant minusDur = instant.minus(Duration.ofMinutes(1));

// 加量单位
Instant plusMins = instant.plus(5, ChronoUnit.MINUTES);

// 加秒 / 毫秒 / 纳秒 (可减)
Instant plusSecs = instant.plusSeconds(30);
Instant plusMillis = instant.plusMillis(500);
Instant plusNanos = instant.plusNanos(100000000L);
```

---

# 📚 参考内容

