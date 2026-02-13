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

通过设置 `-Xlog:gc*=debug` 观察日志, 日志在暂停期间和暂停之外提供了关于垃圾回收活动的详细概述.

### 3.1	避免频繁的 Full GC 

一次完整的 Full GC 非常耗时, 由于老年代堆占用过高引起的 Full GC 可以通过查找关键字 "Pause Full (Allocation Failure)" 来检测, 通常在这之前会出现遇到疏散失败的垃圾回收, 关键字为: `to-space exhausted`. 原因是应用程序分配了太多无法快速回收的对象, 并发标记未能及时完成以开始空间回收阶段, 分配许多大对象会增大遇到 Full GC 的概率, 由于这些对象在 G1 中的分配方式, 它们可能占用比预期更多的内存.
	
解决思路是确保并发标记及时完成, 可以通过减少老年代的分配率, 或者给并发标记更多的时间来完成.

- 通过 `gc+heap=info` 确定 Java 堆上巨大对象占用的区域数量, 关键字为 `Humongus regions: X->Y`, Y 给出由巨大对象占用的区域数量. 如果这个数量比老年代占用的区域数量较高, 可以通过 `-XX:G1HeapRegionSize` 来增大区域大小, 日志开头会打印当前区域的大小.
	
- 增大 Java 堆的大小, 这通常会增加标记所需的时间;
	
- 通过显示的设置 `-XX:ConcGCThreads` 来增大并发标记线程的数量;
	
- 将 G1 设置为更早开始标记, G1 会根据早期应用程序的行为自动确定 IHOP (Initiating Heap Occupancy Percent, 启动堆占用百分比) 阈值, 如果应用程序发生变化, 这个预测可能不正确, 有两种选择, 通过修改 `-XX:G1ReservePercent` 来增加自适应 IHOP 计算中使用的缓冲区, 或者使用 `-XX:-GUseAdaptiveIHOP` 和 `-XXInitiatingHeapOccupancyPercent` 手动设置 IHOP 来禁用它. 

### 3.2	拆解巨型对象

当分配空间给巨型对象的时候, 即使 Java 堆内存尚未耗尽, 也可能发生 Full GC, 因为 G1 需要找到连续的 Region 分配给大对象.

这种情况下可选的解决方法是增大堆内存或者增大 Region 的大小.

在极端情况下, 即使内存足够 G1 也可能找不到足够的连续空间来分配