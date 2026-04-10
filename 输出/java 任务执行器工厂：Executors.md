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

