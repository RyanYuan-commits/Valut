---
type: permanent
banner: Assets/Banner/pexels-vadimsadovski-33522604-7049988.jpg
---
---

**关键词**: G1, Young GC

---

G1 在逻辑上将 Region 分为年轻代和老年代, 比例不是固定的, 为了满足最大停顿时间限制, G1 会自动调整两者的比例.

应用程序在运行时, 会不断创造对象到 Eden 区 , 当 Eden 区空间耗尽, G1 会启动一次 Young GC, 停止应用程序的运行, 回收垃圾对象, 同时可能会触发对象的代晋升, 将对象移动至 Survivor 区或奥年代.



---