---
type: permanent
banner: 附件/Banner/pexels-walidphotoz-1509582.jpg
---
## 1	任务提交方法

```java
// java.util.concurrent.ThreadPoolExecutor#execute
public void execute(Runnable command) {
	// 空任务校验
    if (command == null) throw new NullPointerException();
    int c = ctl.get(); // 描述线程池状态的重要变量
    if (workerCountOf(c) < corePoolSize) {
	    // (1) 当前线程数小于核心线程数，需要创建线程
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // (2) 放入阻塞队列 workQueue.offer(command)
    if (isRunning(c) && workQueue.offer(command)) {
	    // corePoolSize 满了 且 任务成功加入任务队列
        int recheck = ctl.get();
        if (!isRunning(recheck) && remove(command))
	        // 如果线程池不处于运行状态，拒绝策略
            reject(command);
        else if (workerCountOf(recheck) == 0)
	        // 如果工作线程为 0，增加线程
            addWorker(null, false);
    } else if (!addWorker(command, false))
	    // (3)尝试添加非核心线程, 但已经达到最大线程数, 所以添加失败
	    // 执行拒绝策略
        reject(command);
}
```

- 当一个任务被提交到线程池时，首先检查当前线程数是否达到最大线程数；
	- 如果没有，创建一个线程并将该任务作为首个任务交给新线程执行；
	- 如果当前线程数大于最大线程数，将任务放置到阻塞队列（END）；
- 如果放入阻塞队列失败（阻塞队列已满），则尝试添加新的线程，将任务交给新线程执行（END）；
- 如果添加新线程失败（已达到最大线程数），则执行拒绝策略（END）。

---

当前线程池的状态和线程数量，是从 `ctl` 中获取的，`ctl` 是 `ThreadPoolExecutor` 的成员变量，高位用于存储线程池状态，低位用于存储活跃线程数。

```java
// java.util.concurrent.ThreadPoolExecutor#ctl
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
```

---

## 2	任务执行流程

---

==补充内容：ThreadPoolExecutor 的 Worker 内部类==

线程池中的工作线程用 `java.util.concurrent.ThreadPoolExecutor.Worker` 包装，被存放在 `workers` 成员变量中；

```java
private final class Worker extends AbstractQueuedSynchronizer implements Runnable {  
	
    final Thread thread;  
    
    Runnable firstTask;  
     
    volatile long completedTasks;
	
	// ......

}
```

`Worker` 对 `Thread` 的能力进行了拓展，增加了 `completedTasks` 来记录完成任务的数量、`firstTask` 来存储首个执行的任务；

其继承了 `AbstractQueuedSynchronizer` 类，实现了简单的同步锁，同时作为 `Runnbale` 实例，被放置在 `Thread` 对象中运行。

---

==补充内容：ThreadPoolExecutor 的生命周期流转==

![[ThreadPoolExecutor 的生命周期.png]]

`running` 到 `shutdown`：通过 `shutdown()` 方法触发，线程池停止接受新任务，但已提交的任务会执行完成；

`shutdown` 到 `stop`：通过 `shutdownNow()` 方法触发，线程池尝试停止所有正在执行的任务，并清空任务队列中所有待执行的任务；

`shutdown` 到 `tidying`：自然流转，阻塞队列为空，且线程池中工作线程数量为 0，进入 `tidying` 状态；

`tidying` 到 `terminated`：自然流转，当资源被清理完成，执行 `terminated()` 方法，进入 `terminated` 状态。

---

```java
// java.util.concurrent.ThreadPoolExecutor.Worker#run
public void run() {  
    runWorker(this);  
}

// java.util.concurrent.ThreadPoolExecutor#runWorker
final void runWorker(Worker w) {
	Thread wt = Thread.currentThread();
	Runnable task = w.firstTask;
	w.firstTask = null;
	w.unlock();
	// 标识任务完成过程中是否出现异常
	boolean completedAbruptly = true; 
	try {
		// 不断调用 getTask() 方法获取任务
 		while (task != null || (task = getTask()) != null) {
			w.lock();
			if ((runStateAtLeast(ctl.get(), STOP) ||
				 (Thread.interrupted() &&
				  runStateAtLeast(ctl.get(), STOP))) &&
				!wt.isInterrupted())
				// 如果线程池正在停止，确保当前线程被中断；
				// 如果线程池没有停止，确保当前线程没有被中断；
				// 为了处理第二种情况中清除中断状态时可能发生的 shutdownNow 竞态条件，需要进行二次检查。
				wt.interrupt();
			try {
				beforeExecute(wt, task);
				Throwable thrown = null;
				try {
					task.run();
				} catch (RuntimeException x) {
					thrown = x; throw x;
				} catch (Error x) {
					thrown = x; throw x;
				} catch (Throwable x) {
					thrown = x; throw new Error(x);
				} finally {
					afterExecute(task, thrown);
				}
			} finally {
				task = null;
				w.completedTasks++;
				w.unlock();
			}
		}
		completedAbruptly = false;
	} finally {
		processWorkerExit(w, completedAbruptly);
	}
}
```

