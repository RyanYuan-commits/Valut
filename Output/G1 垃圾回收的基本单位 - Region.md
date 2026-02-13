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
// Minimum region size; we won't go lower than that.
// We might want to decrease this in the future, to deal with small
// heaps a bit more efficiently.
#define MIN_REGION_SIZE  (      1024 * 1024 )
// Maximum region size; we don't go higher than that. There's a good
// reason for having an upper bound. We don't want regions to get too
// large, otherwise cleanup's effectiveness would decrease as there
// will be fewer opportunities to find totally empty regions after
// marking.
#define MAX_REGION_SIZE  ( 32 * 1024 * 1024 )
// The automatic region size calculation will try to have around this
// many regions in the heap (based on the min heap size).
#define TARGET_REGION_NUMBER          2048
void HeapRegion::setup_heap_region_size(size_t initial_heap_size, size_t max_heap_size) {
  uintx region_size = G1HeapRegionSize;
  if (FLAG_IS_DEFAULT(G1HeapRegionSize)) {
    size_t average_heap_size = (initial_heap_size + max_heap_size) / 2;
    region_size = MAX2(average_heap_size / TARGET_REGION_NUMBER,
                       (uintx) MIN_REGION_SIZE);
  }
  int region_size_log = log2_long((jlong) region_size);
  // Recalculate the region size to make sure it's a power of
  // 2. This means that region_size is the largest power of 2 that's
  // <= what we've calculated so far.
  region_size = ((uintx)1 << region_size_log);
  // Now make sure that we don't go over or under our limits.
  if (region_size < MIN_REGION_SIZE) {
    region_size = MIN_REGION_SIZE;
  } else if (region_size > MAX_REGION_SIZE) {
    region_size = MAX_REGION_SIZE;
  }
}
```


---