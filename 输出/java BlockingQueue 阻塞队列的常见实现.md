---
type: permanent
banner: 附件/Banner/pexels-8kspain-21564213.jpg
aliases:
  - java 线程池任务队列实现
---
BlockingQueue 是 java 并发编程包中的核心接口，是实现生产者-消费者模型的最佳选择，java 线程池中使用 BlockingQueue 实例作为存放任务的容器；

区别于普通的队列，`BlockingQueue` 的核心特性在于阻塞，当队列为空时，尝试获取元素的线程会被阻塞，直到队列中有元素可取；当队列已满时，尝试添加元素的线程也会被阻塞，直到队列中有剩余空间；

---

常见的阻塞队列的实现有五种：

- `ArrayBlockingQueue`：基于**数组**的**固定大小**的阻塞队列，可以限制任务数量的无限增长；
- `LinkedBlockingQueue`：基于**链表**的容量可选阻塞队列，默认容量为 `Integer.MAX_VALUE`，适合任务处理速度波动较大时使用；
- `SynchronousQueue`：不额外存储元素，当队列中有元素时，`put()` 方法会被阻塞，直到其他线程从队列中取出元素；
- `PriorityBlockingQueue`：**无界**阻塞队列，提供按照优先级排序的能力，排序基于 `Comparator` 接口；
- `DelayQueue`：**无界**阻塞队列，插入时可以**指定过期时间**，元素必须等待过期时间后才能被取出。