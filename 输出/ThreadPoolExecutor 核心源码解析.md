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

`Worker` 在运行过程中会不断调用 `getTask()` 方法，从任务队列中获取可执行任务，然后调用 `run()` 方法执行。

---

==`runWorker()` 方法对于线程池状态的响应==

```java
if ((runStateAtLeast(ctl.get(), STOP) ||
	 (Thread.interrupted() &&
	  runStateAtLeast(ctl.get(), STOP))) &&
	!wt.isInterrupted())
	wt.interrupt();
```

`if` 块中的代码会在线程池处于关闭中状态，且线程没有中断标记时执行，为当前线程打上中断标志；

而如果线程池没有处于关闭中状态，`Thread.interrupted()` 会被调用以清除中断标志，为了防止在调用 `Thread.interrupted()` 方法的同时，线程池变为关闭中状态，需要在清除后再次检查线程池状态。

---

==线程池对于任务执行过程中异常的处理==

`processWorkerExit()` 方法对于正常结束循环的线程和抛出异常结束循环的线程，处理方式是不同的，`runWorker()` 方法通过 `completedAbruptly` 标记来向该方法透传线程结束循环的状态；

`completedAbruptly` 的初始值为 `true` 表示抛出异常退出，当线程正常结束循环退出时，会执行 `completedAbruptly = false` 语句，而如果在执行过程中出现异常，`Worker` 会使用 `thrown` 成员变量记录异常，然后抛出到最外层 `try-finally` 快，直接执行 `processWorkerExit()` 方法执行。

---

## 3	Worker 从阻塞队列中获取任务

```java
private Runnable getTask() {  
    boolean timedOut = false; // 上一次 poll() 是否超时
  
    for (;;) {  
        int c = ctl.get();  
        int rs = runStateOf(c);  

        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {  
            decrementWorkerCount();  
            return null;  
        }  
  
        int wc = workerCountOf(c);  
  
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;  
  
        if ((wc > maximumPoolSize || (timed && timedOut))  
            && (wc > 1 || workQueue.isEmpty())) {  
            if (compareAndDecrementWorkerCount(c))  
                return null;  
            continue;  
        }  
  
        try {  
            Runnable r = timed ?  
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :  
                workQueue.take();  
            if (r != null)  
                return r;  
            timedOut = true;  
        } catch (InterruptedException retry) {  
            timedOut = false;  
        }  
    }  
}
```

`getTask()` 方法用于从阻塞队列获取可执行任务；同时，`getTask()` 方法返回 `null` 是工作线程**正常结束**的唯一途径，所以该方法对于工作线程的状态控制也及其重要。

---

==非核心线程在指定时间时间空闲后会被销毁==

在创建线程池时，我们需要指定 `keepAliveTime` 来控制**非核心**线程在持续空闲多久后可以被销毁，如果发现当前线程数大于核心线程数（或者核心线程也允许在指定时间空闲后被销毁），`timed` 标志会被设置为 `true`，此时从阻塞队列中拉取任务时，会带上 `keepAliveTime` 作为超时时间，如果在超时时间内没有获取到任务，会在下一个循环中尝试销毁该线程（方法返回 `null`）。

```java
// 在下个循环中尝试销毁线程
boolean timed = allowCoreThreadTimeOut || wc > corePoolSize; // 再次判断 

if ((wc > maximumPoolSize || (timed && timedOut))  
	&& (wc > 1 || workQueue.isEmpty())) {  
	if (compareAndDecrementWorkerCount(c))  
		// 尝试销毁
		return null;  
	continue;  
}
```

---

==`getTask()` 方法对于线程池关闭的响应==

```java
if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {  
	decrementWorkerCount();  
	return null;  
}
```

当线程池状态为 shutdown 时，直接返回 `null`，无需再处理剩余任务；而在线程池状态为 stop 时，需要确保任务队列为空。

---

## 4	处理线程退出

```java
// java.util.concurrent.ThreadPoolExecutor#processWorkerExit
private void processWorkerExit(Worker w, boolean completedAbruptly) {  
    if (completedAbruptly) // 如果是异常退出，线程数量还未减一
        decrementWorkerCount();  
  
    final ReentrantLock mainLock = this.mainLock;  
    mainLock.lock();  
    try {  
	    // 统计完成的任务数量
        completedTaskCount += w.completedTasks;  
        workers.remove(w);  
    } finally {  
        mainLock.unlock();  
    }  
  
    tryTerminate();  
  
    int c = ctl.get();  
    if (runStateLessThan(c, STOP)) {  
        if (!completedAbruptly) {
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;  
            if (min == 0 && !workQueue.isEmpty())
                min = 1;
            if (workerCountOf(c) >= min)
                return;
        }  
        addWorker(null, false);  
    }  
}
```

若线程正常结束循环，`completedAbruptly == false`，统计其完成的任务数量，将 `worker` 从集合中移除，方法正常返回，线程退出；

若线程抛出异常，`completedAbruptly == true`，则会在处理完成后，创建一个新的线程。

---

==补充知识：Thread 的异常处理器机制==

对于抛出异常而结束循环的线程，线程池没有复用它，而是创建一个新的线程来取代，是为了适配线程的异常处理器功能；

```java
// java.lang.Thread#dispatchUncaughtException
private void dispatchUncaughtException(Throwable e) {
	getUncaughtExceptionHandler().uncaughtException(this, e);
}
```

在创建线程后，通过 `setUncaughtExceptionHandler()` 方法来为线程指定一个自定义的异常处理器，处理方法会被 jvm 调用。

---