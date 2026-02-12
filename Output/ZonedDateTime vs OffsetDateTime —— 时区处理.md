---
type: permanent
---
---

**关键词**: java, 时区处理, ZoneDateTime, OffsetDateTime

---

## 1	偏移量 vs 时区

**偏移量**: 仅仅是一个数字, 表示**当前时间**与 **UTC**/格林威治时间的**差值**. 例如 +08:00. 它**不包含任何地理或历史规则**, 在 Java 中使用 `ZoneOffset` 表示;

**时区**: 是一个**地理区域**的标识, 如 `Asia/Shanghai` 或 `America/New_York`. 时区 ID 包含了偏移量, 但它不仅仅是偏移量. 它背后拥有一套完整的 "规则库"(TZDB), 记录了这个地区在历史上什么时候切换过时区, 以及什么时候实行**夏令时 (DST)**, 在 Java 中使用 `ZoneId` 表示.

## 2	ZonedDateTime vs OffsetDateTime

`OffsetDateTime` = `LocalDateTime` + `ZoneOffset`, 代表了一个绝对的时刻加上一个固定的偏移量, 它只知道此时此刻比 UTC 快或慢多久, 对机器极其友好;

`ZonedDateTime` = `LocalDateTime` + `ZoneId` (+ `ZoneOffset`), 代表了一个绝对的时刻加上一个具有地理属性的时区, 能自动感知夏令时切换. 

假设在纽约 (America/New_York), 夏令时切换的那一刻, 时钟会从 01:59 直接跳到 03:00, 如果用 `ZonedDateTime` 加 1 小时, 它会知道现在是 03:00. 如果用 `OffsetDateTime`, 它只会简单地根据数字加减, 可能导致逻辑上的混乱.

## 3	创建方法和常用 API

### 3.1	创建方式

```java
// 获取当前系统时区的时间
ZonedDateTime zdtNow = ZonedDateTime.now();
OffsetDateTime odtNow = OffsetDateTime.now();

// 指定具体时区/偏移量
ZonedDateTime zdt = ZonedDateTime.of(LocalDateTime.now(), ZoneId.of("Asia/Shanghai"));
OffsetDateTime odt = OffsetDateTime.of(LocalDateTime.now(), ZoneOffset.ofHours(8));

// 从 Instant 转化
ZonedDateTime zdtFromInstant = Instant.now().atZone(ZoneId.of("Europe/Paris"));
```

### 3.2	常用操作

**转换时区**(保持同一瞬间): `zdt.withZoneSameInstant(ZoneId. of("UTC"))`

**修改字段**: `odt.withHour(10).plusDays(1)`

**获取偏移量**: `zdt.getOffset()`, (注意: `ZonedDateTime` 的偏移量是根据日期自动计算出来的)

## 4	转换关系

![[java time 包转化关系.png]]

---