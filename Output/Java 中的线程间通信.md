---
type: permanent
banner: Assets/Banner/pexels-bertellifotografia-1144690.jpg
---
# 🌐 核心观点

线程通信是多个独立运行的线程之间交换数据或协调执行顺序的机制. 在 Java 中, 它主要通过**共享内存**配合**信号通知**来实现.

---

# 🔖 详细解释

线程通信指的是多个线程之间为了协调完成某个共同任务, 而以某种机制来交换信息, 传递状态, 协调行动的过程.

## 1	线程通信分层架构

![[Java 线程通信的分层设计.png|600]]
Java 线程间通信遵循分层设计, 每层建立在下层原语之上:

- 硬件级: CPU 缓存一致性协议和内存屏障确保跨核心缓存同步
	​
- 操作系统级: 互斥锁, 信号量, 上下文切换等系统调用
	
- Java 语言级: synchronized, volatile, Monitor 机制
	
- 并发框架: java.util.concurrent 包中的高级抽象
	
- 虚拟线程: Java 19+ 引入的轻量级线程模型

## 2	基础通信机制

### 2.1	wait/notify 模式

提供了 wait, notify, notifyAll 方法来实现线程同步:

- wait 使当前线程释放对象的监视器锁并进入阻塞状态
	
- notify 唤醒单个等待线程, notifyAll 唤醒所有等待线程.

上述方法必须在 synchronized 块中调用, 否则抛出 IllegalMonitorStateException.

### 2.2	synchronized 同步块

synchronized 提供互斥锁, 保证临界区的原子性, 有同步方法和同步块两种形式:

```java
public synchronized void send(String packet) {
	// do something
}

public void send(String packet) {
    synchronized(lock) {
		// do something
    }
}
```

## 3	并发框架

### 3.1	BlockingQueue 线程安全队列

BlockingQueue 是生产者-消费者模式的企业级实现, 内置 wait/notify 逻辑.

- 当队列为空时, take 方法自动阻塞消费者;
	
- 当队列满时, put 方法自动阻塞生产者;
	
- 内部基于 ReentrantLock + Condition 实现.

常用的实现有:

| 实现                    | 特点             | 应用         |
| --------------------- | -------------- | ---------- |
| ArrayBlockingQueue    | 数组实现, 单锁, 固定容量 | 有界队列, 容量确定 |
| LinkedBlockingQueue   | 链表实现, 双锁分离     | 高吞吐, 容量较大  |
| SynchronousQueue      | 零容量，直接交付       | 线程间直接交换    |
| PriorityBlockingQueue | 优先级队列          | 需排序处理      |
| DelayQueue            | 延迟队列           | 定时任务       |

### 3.2	ExecutorService 线程池

是现代 Java 应用的标准, 通过内置队列实现任务和线程的解耦.

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
for (Task task : tasks) {
    executor.submit(task);
}
executor.shutdown();
```

### 3.3	CountDownLatch 等待多任务完成

CountDownLatch 用于等待 N 个独立任务全部完成, 初始化一个计数器, 每个任务完成时 countDown, 主线程 await 阻塞直到计数为 0.

```java
int workerCount = 4;
CountDownLatch latch = new CountDownLatch(workerCount);
ExecutorService executor = Executors.newFixedThreadPool(workerCount);

// 启动采集线程
for (int i = 0; i < workerCount; i++) {
    final int id = i;
    executor.submit(() -> {
        try {
            System.out.println("Worker " + id + " collecting...");
            Thread.sleep(ThreadLocalRandom.current().nextInt(1000, 5000));
            System.out.println("Worker " + id + " done");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            latch.countDown();  // 每个任务完成时递减
        }
    });
}

System.out.println("Waiting for all workers...");
latch.await();  // 阻塞直到计数到零
System.out.println("All workers completed");
executor.shutdown();
```

一次性使用, 不可复用.

### 3.4	CyclicBarrier 循环同步点

让多个线程在同步点相互等待, 所有线程到达后一起继续, 可以同步使用.

```java
int seats = 27;
ExecutorService executor = Executors.newFixedThreadPool(seats);

// 所有乘客就座后，关闭安全杆
CyclicBarrier barrier = new CyclicBarrier(seats, () -> {
    System.out.println("All passengers seated, closing bar");
});

CountDownLatch startLatch = new CountDownLatch(seats);

for (int i = 1; i <= seats; i++) {
    final int person = i;
    executor.submit(() -> {
        try {
            System.out.println("Person " + person + " taking seat");
            Thread.sleep(1500);
            barrier.await();  // 等待所有人上车
            System.out.println("Person " + person + " ready");
            startLatch.countDown();
        } catch (Exception e) {
            e.printStackTrace();
        }
    });
}

startLatch.await();
System.out.println("Start rollercoaster!");
executor.shutdown();
```

### 3.5	Semaphore 资源限流

Semaphore 控制同时访问某个资源的线程数.

```java
Semaphore semaphore = new Semaphore(3);  // 最多 3 个并发连接

ExecutorService executor = Executors.newFixedThreadPool(10);

for (int i = 0; i < 10; i++) {
    final int id = i;
    executor.submit(() -> {
        try {
            semaphore.acquire();  // 获得许可
            System.out.println("Thread " + id + " acquired");
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } finally {
            semaphore.release();  // 释放许可
            System.out.println("Thread " + id + " released");
        }
    });
}
```

## 4	并发容器



---

# 📚 参考内容

