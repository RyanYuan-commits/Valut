---
type: permanent
banner: 附件/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
`java.util.concurrent.Executors` 提供了一系列静态工厂方法，用于创建不同类型 `ExecutorService` 的工厂类；

---

```java
// java.util.concurrent.Executors#newFixedThreadPool
public static ExecutorService newFixedThreadPool(int nThreads) {  
    return new ThreadPoolExecutor(nThreads, nThreads,  
                                  0L, TimeUnit.MILLISECONDS,  
                                  new LinkedBlockingQueue<Runnable>());  
}
```

创建一个大小固定的线程池，核心线程数和最大线程数相等，阻塞队列是无界队列；

---

```java
// java.util.concurrent.Executors#newSingleThreadExecutor()
public static ExecutorService newSingleThreadExecutor() {  
    return new FinalizableDelegatedExecutorService  
        (new ThreadPoolExecutor(1, 1,  
                                0L, TimeUnit.MILLISECONDS,  
                                new LinkedBlockingQueue<Runnable>()));  
}
```

使用 Executor 包装单个线程，提交的任务会按照提交顺序执行，适合需要确保任务顺序的场景；将线程管理交给线程池，在发生异常时，线程池会自动创建一个线程来替代。

---

```java
public static ExecutorService newCachedThreadPool() {  
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,  
                                  60L, TimeUnit.SECONDS,  
                                  new SynchronousQueue<Runnable>());  
}
```

核心线程数为 0，同步队列不存储任务，每个被提交的任务都会分配新线程执行，最大线程数为 `Integer.MAX_VALUE`，线程最大存活时间为 60s；

适合执行大量短期的异步任务，但是如果短期内提交的任务过多，会导致线程数量过多，导致系统资源的耗尽。

---

```java
public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory) {  
    return new DelegatedScheduledExecutorService  
        (new ScheduledThreadPoolExecutor(1, threadFactory));  
}
```

用于支持定时和周期性任务的执行，提供了 `schedule()`、`scheduleAtFixedRate()` 等方法。

---