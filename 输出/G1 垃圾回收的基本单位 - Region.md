---
type: permanent
banner: Assets/Banner/pexels-vadimsadovski-33522604-7049988.jpg
---
---

**关键词**: gc, g1

---

传统的 GC 收集器将连续的内存空间划分为新生代, 老年代和永久代 (JDK8 去除了永久代, 引入元空间), 而 G1 的各代存储地址是不连续的, 每一代都使用 n 个不连续的, 大小相同的 Region, 每个 Region 占有一块连续的虚拟内存地址.

![[g1 GC 内存布局.png]]

除了新生代, 老年代, Survivor 区域之外,  G1 还设置了 Humongous 大对象区, 用于存放大小超过 Region 大小一半的对象, 这些对象有如下的特点:

- 大对象直接被分配到了老年代, 防止反复拷贝移动;
	
- 大对象在 Global Concurrent Marking 阶段的 Cleanup 和 Full GC 阶段回收;
	
- 在分配大对象之前会检查是否超过了初始化堆占用百分比和标记阈值, 若超过则启动 Global Concurrent Marking, 为的是提早回收, 防止疏散失败和 Full GC.

Region 的大小通过参数 `-XX:G1HeapRegionSize` 设置, 取值范围从 1M 到 32M, 且是 2 的指数, 如果不设置, G1 会根据 Heap 大小自动决定:

```cpp
// share/vm/gc_implementation/g1/heapRegion.cpp
// 最小 Region 大小；我们不会设置比这更小的值。
// 未来我们可能考虑减小此值，以便更有效地处理较小的堆内存。
#define MIN_REGION_SIZE  (      1024 * 1024 )

// 最大 Region 大小；我们不会设置比这更大的值。设置上限有充分的理由：
// 我们不希望 Region 过大，否则在标记完成后，寻找完全空闲 Region 的机会将减少，
// 从而降低清理效率。
#define MAX_REGION_SIZE  ( 32 * 1024 * 1024 )

// 自动 Region 大小计算将尝试在堆中维持大约这个数量的 Region（基于最小堆大小）。
#define TARGET_REGION_NUMBER          2048

void HeapRegion::setup_heap_region_size(size_t initial_heap_size, size_t max_heap_size) {
  uintx region_size = G1HeapRegionSize;
  if (FLAG_IS_DEFAULT(G1HeapRegionSize)) {
    // 若未手动设置 G1HeapRegionSize 参数，则自动计算
    size_t average_heap_size = (initial_heap_size + max_heap_size) / 2;
    region_size = MAX2(average_heap_size / TARGET_REGION_NUMBER,
                       (uintx) MIN_REGION_SIZE);
  }
  int region_size_log = log2_long((jlong) region_size);
  // 重新计算 Region 大小，确保其为 2 的幂。
  // 这意味着 region_size 是不超过当前计算值的最大 2 的幂。
  region_size = ((uintx)1 << region_size_log);
  
  // 确保最终值在最小/最大限制范围内
  if (region_size < MIN_REGION_SIZE) {
    region_size = MIN_REGION_SIZE;
  } else if (region_size > MAX_REGION_SIZE) {
    region_size = MAX_REGION_SIZE;
  }
}
```

常用的关于 Region 的日志:

- `[debug] gc.heap`: 打印 Region 的大小, 以及各个分代各自占用的大小, 如 `region size 8192K, 921 young (7544832K), 6 survivors (49152K)`.
	
- `[info] gc.heap`: 查看某次 GC 堆中每种区域的变化, 如 `Eden regions: 915->0(915)`, `Humongous regions: 2->2`, 括号里的数字表示当前这个分代的 Region 数.

---