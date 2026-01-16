---
type: permanent
banner: Assets/Banner/pexels-eliannedipp-4666748.jpg
---
# 🌐 核心观点

在 `java.time` API 中, 时间量度被拆分为 Duration 和 Period 两个维度.

---

# 🔖 详细解释

## 1	核心定义

在 java. time API 中, 时间量度被严谨地拆分为两个维度:

- **Duration**: 表示**物理时长**, 基于时间轴, 衡量的是 "**物理上流逝了多少秒**", 是精密, 连续的机器视角.
	
- **Period**:  表示**逻辑步长**, 基于日历. 衡量的是“**人类直觉中的年月日**”, 是离散, 具有文化属性的人类视角.

## 2	Duration: 精密计算的标尺

`Duration` 内部通过 `seconds` 和 `nanos` 两个字段存储, 可以用于操作 `Instant`, `LocalTime`, `LocalDateTime`, `ZonedDateTime`.

```java
// java.time.Duration
public final class Duration implements TemporalAmount, Comparable<Duration>, Serializable {
		
    // ...
	
	/**
	 * The number of seconds in the duration.
	 */
	private final long seconds;
	
	/**
	 * The number of nanoseconds in the duration, expressed as a fraction of the
	 * number of seconds. This is always positive, and never exceeds 999,999,999.
	 */
	private final int nanos;

	// ...
	
}
```

Duration 不包含时间概念, 只计算两个时间点之间的绝对距离.

```java
Instant start = Instant.now();

// ......

Instant end = Instant.now();
Duration elapsed = Duration.between(start, end);
System.out.println("耗时：" + elapsed.toMillis() + "ms");
```

## 3	Period: 日历逻辑的跨度

`Period` 内部通过 `years`, `months` 和 `days` 三个 `int` 字段存储, 仅用于操作 `LocalDate`.

```java
LocalDate birthday = LocalDate.of(1995, 5, 20);
LocalDate today = LocalDate.now();
Period age = Period.between(birthday, today);
System.out.printf("工龄/年龄：%d年%d个月%d天", age.getYears(), age.getMonths(), age.getDays());
```

是 "名义上" 的跨度. 例如, `Period.ofMonths(1)` 加在 1 月 31 日上, 结果会是 2 月 28/29 日.


---

# 📚 参考内容

