---
type: permanent
banner: 附件/Banner/pexels-bertellifotografia-1144690.jpg
---
若任务队列已满，且线程池内的线程数达到最大线程数，会触发拒绝策略：

```java
// java.util.concurrent.ThreadPoolExecutor#execute
if (isRunning(c) && workQueue.offer(command)) {  
    ......
}  
else if (!addWorker(command, false))  
	// 触发拒绝策略
    reject(command);

// java.util.concurrent.ThreadPoolExecutor#reject
final void reject(Runnable command) {  
    handler.rejectedExecution(command, this);  
}
```

在 java concurrent 包中，拒绝策略使用 `RejectedExecutionHandler` 接口表示，在 `ThreadPoolExecutor` 中，提供了一些拒绝策略的实现：

- `AbortPolicy`：抛出 `RejectedExecutionException` 异常；
- `CallerRunsPolicy`：由提交任务的线程来执行该任务；
- `DiscardOldestPolicy`：丢弃掉阻塞队列中最旧的任务，然后尝试重新提交任务；
- `DiscardPolicy`：直接丢弃掉任务。

如果想要任务**全部被执行**，在不引入其他策略的情况下，只能选择 `CallerRunsPolicy`。

