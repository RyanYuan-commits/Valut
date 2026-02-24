---
type: MOC
banner:
---
G1 提供了两种 GC 模式, Young GC 和 Mixed GC, 两种都是完全 Stop The World 的. 

- [[G1 中的 Young GC|Young GC]]: 选定所有年轻代里的 Region. 通过控制年轻代的 region 个数, 即年轻代内存大小, 来控制 young GC 的时间开销. 
	
- [[G1 中的 Mixed GC|Mixed GC]]: 选定所有年轻代里的 Region, 外加根据 global concurrent marking 统计得出收集收益高的若干老年代 Region. 在用户指定的开销目标范围内尽可能选择收益高的老年代 Region.