---
source: https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-garbage-collector-tuning.html#GUID-90E30ACA-8040-432E-B3A0-1E0440AB556A
type: input
banner: "[[pexels-vadimsadovski-33522604-7049988.jpg]]"
---
## 1	G1 的一般建议

G1 默认配置的目标是: 在高吞吐量的情况下提供相对较小切均匀的暂停.

- 如果注重高吞吐量, 可以使用 `-XX:MaxGCPauseMillis` 来放松暂停时间目标或提供更大的堆;
	
- 如果注重延迟, 则调低 `-XX:MaxGCPauseMillis` 参数.

避免通过 `-Xmn`, `-XX:NewRatio` 和其他选项来限制年轻代的大小, 因为年轻代大小是 G1 满足暂停时间目标的主要手段, 将年轻带大小设置为一个固定值会覆盖并实际上禁用暂停时间控制.

## 2	从其他收集器迁移到 G1

移除所有影响垃圾回收的选项，仅设置暂停时间, 堆大小, 可选择性的设置 `-Xms`;

许多对于其他收集器有用的选项对于 G1 可能完全不起作用, 如设置年轻代大小可能会完全组织 G1 调整年轻代大小来满足暂停时间的要求.

## 3	提高 G1 性能

应用级别的优化更有效, 如减少寿命较短的对象的数量;

通过 `-Xlog:gc*`
