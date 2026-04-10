---
type: permanent
banner: 附件/Banner/pexels-clive-kim-2523249-4220967.jpg
---
- `corePoolSize`：核心线程数，被创建后默认不会被销毁；
- `maximumPoolSize`：线程池中允许存在的最大线程数，如果当前线程池中的线程数已达到了核心线程数，并且任务队列已满，则会创建新的线程。
- `keepAliveTime`：当非核心线程无任务执行时，其能存活的最大时间；
- `timeUnit`：`keepAliveTime` 的时间单位；
- `workQueue`：[[java BlockingQueue 阻塞队列的常见实现|任务队列]]，用于存放等待执行的任务的队列；
- `threadFactory`：创建新线程的线程工厂，通常用于额外定义线程的名称、优先级等；
- `handler`：[[java 线程池的拒绝策略|拒绝策略]]，若在线程池已满时仍然添加新任务，应当有什么逻辑处理；
- `allowCoreThreadTimeOut`：是否允许 `keepAliveTime` 作用于核心线程。