---
type: permanent
banner: Assets/Banner/pexels-maxravier-3331094.jpg
---
---

**关键词**: G1, 调优

---

## 1	诱发原因

- 疏散失败 (最核心原因): 有年轻代晋升失败和老年代转移失败两种情况;

- 巨型对象分配失败: 类似老年代转移失败, 老年代中没有足够的空间分配给新的巨型对象;
	
- 并发模式失败: 在并发标记过程尚未完成时, 老年代的空间就已耗尽 (如年轻代大量晋升);
	
- 元空间不足: 加载的类过多 (如大量使用动态代理, CGLib, JSP 等), 导致增长到阈值且无法扩容;
	
- 显示调用 `System.gc()`

## 2	观测 Full GC

关键字 `Pause Full (Allocation Failure)` 表明出现了 Full GC. 

在这之前通常会遇到疏散失败的垃圾回收, 关键字为 `to-space exhausted`, 原因是应用程序分配了太多无法快速回收的对象, 并发标记未能及时完成以开始空间回收阶段. 分配许多大对象会增大遇到 Full GC 的概率, 由于这些对象在 G1 中的分配方式, 它们可能占用比预期更多的内存.

## 3	调优方式

解决思路是**确保并发标记及时完成**, 可以通过减少老年代的分配率, 或者给并发标记更多的时间来完成.

- 通过 `gc+heap=info` 确定 Java 堆上巨大对象占用的区域数量, 关键字为 `Humongus regions: X->Y`, Y 给出由巨大对象占用的区域数量. 如果这个数量比老年代占用的区域数量较高, 可以通过 `-XX:G1HeapRegionSize` 来增大区域大小, 日志开头会打印当前区域的大小.
	
- 增大 Java 堆的大小, 这通常会增加标记所需的时间;
	
- 通过显示的设置 `-XX:ConcGCThreads` 来增大并发标记线程的数量;
	
- 将 G1 设置为更早开始标记, G1 会根据早期应用程序的行为自动确定 IHOP (Initiating Heap Occupancy Percent, 启动堆占用百分比) 阈值, 如果应用程序发生变化, 这个预测可能不正确, 可以通过修改 `-XX:G1ReservePercent` 来增加自适应 IHOP 计算中使用的缓冲区, 或使用 `-XX:-GUseAdaptiveIHOP` 和 `-XXInitiatingHeapOccupancyPercent` 手动设置 IHOP 来禁用它. 

---