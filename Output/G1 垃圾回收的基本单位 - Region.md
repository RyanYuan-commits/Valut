---
type: permanent
banner: Assets/Banner/pexels-vadimsadovski-33522604-7049988.jpg
---
---

**关键词**: gc, g1

---

传统的 GC 收集器将连续的内存空间划分为新生代, 老年代和永久代 (JDK8 去除了永久代, 引入元空间), 而 G1 的各代存储地址是不连续的, 每一代都使用 n 个不连续的, 大小相同的 Region, 每个 Region 占有一块连续的虚拟内存地址:

![[g1 GC 内存布局.png]]



---