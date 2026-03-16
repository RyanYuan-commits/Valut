---
type: permanent
banner: 附件/Banner/pexels-vadimsadovski-33522604-7049988.jpg
aliases:
  - CAP
---
---

**关键词**: 分布式，CAP

---

![[CAP 理论.png|400]]

 CAP 理论指的是：对于一个系统，一致性，可用性和分区容错性只能满足其二。

- 一致性（Consistency）：强调数据的正确性，每次读操作，要么读到最新的数据，要么失败，要求整个分布式系统像一个不可拆分的整体，写操作具有原子性。
- 可用性（Availability）：强调服务的使用体验，每次请求不会失败且不会等待时间过长。
- 分区容错性（Partition Tolerance）：在网络不可靠的背景下，系统不会出现不可用的情况。



---