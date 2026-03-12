---
type: permanent
banner: Assets/Banner/pexels-picjumbo-com-55570-4457409.jpg
---
---

**关键词**: G1, Mixed GC

---

当堆占用达到 IHOP 时, 会触发全局标记进而触发 Mixed GC, 回收所有的新生代和部分老年代, 如果在复制过程中发现没有足够的空 Region 放置对象, 会触发一次 Full GC. 主要分为全局并发标记和拷贝存活对象两个步骤.

## 1	收集过程

### 1.1	全局并发标记过程

IHOP 默认值为 45, 含义是老年代使用空间 / 整个堆空间的比例, 当达到这个比例的时候, G1 并不会立刻触发全局标记, 而是会等待下一次 Young GC, 利用年轻代收集的 STW 阶段完成初始标记. 全局并发标记并不是一次 GC 的必须环节, 通常是**多次执行** Young GC 后才会触发一次, 主要分为五个步骤:

- **初始标记**: 整个过程 STW, 标记了从 GC Root 直接可达的对象, 会发生 stop the world, 但是该阶段耗时很短;
- **根区域扫描**: 标记出仅被年轻代存活对象所引用的老年代对象, 并将这些老年代对象作为**并发标记**阶段扫描老年代的根的一部分, 避免这部分对象被当作垃圾回收;
- **并发标记**: 与应用程序并发执行, 从 GC Roots 开始对堆中的对象进行可达性分析;
- **重新标记**: 处理在并发标记过程中发生引用变动的那部分对象, 采用的是初始快照方法;
- **清除**: 计算每个 Region 中存活的对象, 把无存活对象的 Region 直接放到空闲列表中, 该阶段还会重置 Remember Set, 并对每个 Region 排序, 估算 Region 的回收成本, 构建最符合用户期望的停顿时间的回收计划.

### 1.2	并发标记后

并发标记结束以后, 老年代中全部为垃圾的 Region 直接被回收, 仅部分为垃圾的 Region 会进行混合回收;

根据停顿目标, G1 可能没法一次性回收掉所有的老年代候选分区, 只能选择优先级高的若干个 Region 进行回收, 对优先级高的, 回收垃圾性价比高的 Region 被分成 8 次回收(通过 `-XX:G1MixedGCCountTarget` 设置, 默认阈值 8), 垃圾占内存分段比例越高的, 越会被先回收. 由参数 `-XX:G1MixedGCLiveThresholdPercent` (默认 65%)阈值决定内存分段是否被回收;

回收并不一定要进行 8 次, 由参数 `-XX:G1HeapWastePercent` (默认值 10%)控制, `-XX:G1HeapWastePercent` 允许整个堆内存有 10% 的空间浪费, 意味着如果发现可以回收的垃圾占堆内存的比例低于 10%, 则不再进行回收, 这样利用率比较高, 否则 GC 计算完, 回收该区域会花费很多的时间, 但是回收到的内存却很少, 回收性价比不高.

### 1.3	拷贝存活对象

对象复制过程又称为 Evacuation, 是一个 STW 过程. 这一步会复用 Yong GC 的逻辑, 但是会额外选择收益较高的 Old Region. 将 CSet 中的存活对象复制到新的 Region, 如果没有足够的空间则触发 Full GC.

## 2	全局并发标记日志

```java
// 发生全局并发标记的原因是大对象分配
66955.252: [G1Ergonomics (Concurrent Cycles) request concurrent cycle initiation, reason: occupancy higher than threshold, occupancy: 1449132032 bytes, allocation request: 579608 bytes, threshold: 1449
551430 bytes (45.00 %), source: concurrent humongous allocation]
2014-12-10T11:13:09.532+0800: 66955.252: Application time: 2.5750418 seconds
 66955.259: [G1Ergonomics (Concurrent Cycles) request concurrent cycle initiation, reason: requested by GC cause, GC cause: G1 Humongous Allocation]
{Heap before GC invocations=1874 (full 4):
 garbage-first heap   total 3145728K, used 1281786K [0x0000000700000000, 0x00000007c0000000, 0x00000007c0000000)
  region size 1024K, 171 young (175104K), 27 survivors (27648K)
 Metaspace       used 116681K, capacity 137645K, committed 137984K, reserved 1171456K
  class space    used 13082K, capacity 16290K, committed 16384K, reserved 1048576K
 66955.259: [G1Ergonomics (Concurrent Cycles) initiate concurrent cycle, reason: concurrent cycle initiation requested]
2014-12-10T11:13:09.539+0800: 66955.259: [GC pause (G1 Humongous Allocation) (young) (initial-mark)
…….
2014-12-10T11:13:09.597+0800: 66955.317: [GC concurrent-root-region-scan-start]
2014-12-10T11:13:09.597+0800: 66955.318: Total time for which application threads were stopped: 0.0655753 seconds
2014-12-10T11:13:09.610+0800: 66955.330: Application time: 0.0127071 seconds
2014-12-10T11:13:09.614+0800: 66955.335: Total time for which application threads were stopped: 0.0043882 seconds
2014-12-10T11:13:09.625+0800: 66955.346: [GC concurrent-root-region-scan-end, 0.0281351 secs]
2014-12-10T11:13:09.625+0800: 66955.346: [GC concurrent-mark-start]
2014-12-10T11:13:09.645+0800: 66955.365: Application time: 0.0306801 seconds
2014-12-10T11:13:09.651+0800: 66955.371: Total time for which application threads were stopped: 0.0061326 seconds
2014-12-10T11:13:10.212+0800: 66955.933: [GC concurrent-mark-end, 0.5871129 secs]
2014-12-10T11:13:10.212+0800: 66955.933: Application time: 0.5613792 seconds
2014-12-10T11:13:10.215+0800: 66955.935: [GC remark 66955.936: [GC ref-proc, 0.0235275 secs], 0.0320865 secs]
 [Times: user=0.05 sys=0.00, real=0.03 secs]
2014-12-10T11:13:10.247+0800: 66955.968: Total time for which application threads were stopped: 0.0350098 seconds
2014-12-10T11:13:10.248+0800: 66955.968: Application time: 0.0001691 seconds
2014-12-10T11:13:10.250+0800: 66955.970: [GC cleanup 1178M->632M(3072M), 0.0060632 secs]
 [Times: user=0.02 sys=0.00, real=0.01 secs]
2014-12-10T11:13:10.256+0800: 66955.977: Total time for which application threads were stopped: 0.0088462 seconds
2014-12-10T11:13:10.257+0800: 66955.977: [GC concurrent-cleanup-start]
2014-12-10T11:13:10.259+0800: 66955.979: [GC concurrent-cleanup-end, 0.0024743 secs
```

---