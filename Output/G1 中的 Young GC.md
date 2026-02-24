---
type: permanent
banner: Assets/Banner/pexels-vadimsadovski-33522604-7049988.jpg
---
---

**关键词**: G1, Young GC

---

## 1	收集过程

G1 在逻辑上将 Region 分为年轻代和老年代, 比例不是固定的, 为了满足最大停顿时间限制, G1 会自动调整两者的比例. 应用程序在运行时, 会不断创造对象到 Eden 区, **当 Eden 区空间耗尽**, G1 会启动一次 Young GC, 停止应用程序的运行, 回收垃圾对象, 同时可能会触发对象的代晋升, 将对象移动至 Survivor 区或奥年代.

- **选择收集集合**: G1 会在遵循用户设置的 GC 暂停时间上限的基础上, 选择一个最大年轻代区域数, 将这个数量的所有年轻代区域作为收集集合;
	
- **根处理**: 根是指 static 静态变量指向的对象, 正在执行的方法调用链上的局部变量等. 根引用连同记忆集 Rset 记录的外部引用, 作为扫描存活对象的入口;
	
- **更新 RSet 记忆集合**: 处理 dirty card 队列中的 card, 更新 RSet 记忆集合, 此阶段完成后, RSet 记忆集合可以准确的记录年轻代对象对老年代的引用, 标记出老年代对本 Region 分区中的对象的引用
	
- **RSet 扫描 Scan RS**: 根据 Rset 的记录, 识别被老年代对象指向的 Eden 中的对象, 这些有引用, 指向的 Eden 中年轻代对象的, 都被认为是存活的对象
	
- **对象拷贝**: 将上面计算标记出来的 Eden 区存活的对象将被复制拷贝到 survivor to 区, 如果幸存者区内存活的对象的年龄没有达到阈值, 年龄会+1, 如果年龄达到阈值会被复制到老年代未使用的空间中进行分配. 如果幸存者空间不够, Eden 区的部分数据会直接晋升到老年代中
	
- **处理引用**: 处理软引用, 弱引用, 虚引用, 最终 Eden 空间的数据为空, GC 停止工作, 而目标内存中的对象都是连续存储的, 没有碎片, 因此复制过程可以达到内存整理的效果, 减少碎片.

当年轻代处理完成后, 会伴随着老年代的全局并发标记, 为下一步 Mixed GC 提供标记服务, 过程与 CMS 收集器的并发标记类似.

## 2	Young GC 日志

```java
{Heap before GC invocations=12 (full 1):
 // 展示堆区域的使用情况
 garbage-first heap   total 3145728K, used 336645K [0x0000000700000000, 0x00000007c0000000, 0x00000007c0000000)
  region size 1024K, 172 young (176128K), 13 survivors (13312K)
 Metaspace       used 29944K, capacity 30196K, committed 30464K, reserved 1077248K
  class space    used 3391K, capacity 3480K, committed 3584K, reserved 1048576K
  
// 表明 GC 原因是 Young GC
2014-11-14T17:57:23.654+0800: 27.884: [GC pause (G1 Evacuation Pause) (young)
Desired survivor size 11534336 bytes, new threshold 15 (max 15)
- age   1:    5011600 bytes,    5011600 total
 27.884: [G1Ergonomics (CSet Construction) start choosing CSet, _pending_cards: 1461, predicted base time: 35.25 ms, remaining time: 64.75 ms, target pause time: 100.00 ms]
 27.884: [G1Ergonomics (CSet Construction) add young regions to CSet, eden: 159 regions, survivors: 13 regions, predicted young region time: 44.09 ms]
 27.884: [G1Ergonomics (CSet Construction) finish choosing CSet, eden: 159 regions, survivors: 13 regions, old: 0 regions, predicted pause time: 79.34 ms, target pause time: 100.00 ms]
, 0.0158389 secs]
   [Parallel Time: 8.1 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 27884.5, Avg: 27884.5, Max: 27884.5, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 0.4, Avg: 0.8, Max: 1.2, Diff: 0.8, Sum: 3.1]
      [Update RS (ms): Min: 0.0, Avg: 0.3, Max: 0.6, Diff: 0.6, Sum: 1.4]
         [Processed Buffers: Min: 0, Avg: 2.8, Max: 5, Diff: 5, Sum: 11]
      [Scan RS (ms): Min: 0.0, Avg: 0.1, Max: 0.1, Diff: 0.1, Sum: 0.3]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.1, Max: 0.2, Diff: 0.2, Sum: 0.6]
      [Object Copy (ms): Min: 4.9, Avg: 5.1, Max: 5.2, Diff: 0.3, Sum: 20.4]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.4, Max: 1.3, Diff: 1.3, Sum: 1.4]
      [GC Worker Total (ms): Min: 6.4, Avg: 6.8, Max: 7.8, Diff: 1.4, Sum: 27.2]
      [GC Worker End (ms): Min: 27891.0, Avg: 27891.3, Max: 27892.3, Diff: 1.3]
   [Code Root Fixup: 0.5 ms]
   [Code Root Migration: 1.3 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.2 ms]
   [Other: 5.8 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 5.0 ms]
      [Ref Enq: 0.1 ms]
      [Redirty Cards: 0.0 ms]
      [Free CSet: 0.2 ms]
   [Eden: 159.0M(159.0M)->0.0B(301.0M) Survivors: 13.0M->11.0M Heap: 328.8M(3072.0M)->167.3M(3072.0M)]
Heap after GC invocations=13 (full 1):
 garbage-first heap   total 3145728K, used 171269K [0x0000000700000000, 0x00000007c0000000, 0x00000007c0000000)
  region size 1024K, 11 young (11264K), 11 survivors (11264K)
 Metaspace       used 29944K, capacity 30196K, committed 30464K, reserved 1077248K
  class space    used 3391K, capacity 3480K, committed 3584K, reserved 1048576K
}
 [Times: user=0.05 sys=0.01, real=0.02 secs]
```



---