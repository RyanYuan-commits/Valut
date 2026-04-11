---
type: permanent
banner: 附件/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
java 线程池的状态在 `java.lang.Thread.State` 枚举中被定义，共有六种：

- `new`：线程刚刚被创建，还没有调用 `start()` 方法启动；
- `runnable`：涵盖了操作系统层面的 `ready` 和 `running` 状态，在 java 看来，只要线程没有被阻塞或等待，统称为 `runnable`；
- `blocked`：阻塞等待获取监视器锁，仅发生在 `synchronized` 关键字同步方法块或方法，且竞争锁失败时，与 `waitting` 状态做区分；
- `waitting`：线程主动挂起，等待其他线程执行特定操作唤醒或中断，如果没有外部干预，将会无限期暂停；由不带超时时间的： `Object.wait()`、`Thread.join()`、`LockSupport.park()` 方法触发；
- `timed_waitting`：带有超时的等待，由 `Thread.sleep(long)`、带有超时的 `Object.wait()`、`Thread.join()`、`LockSupport.parkNanos()` 触发；
- `terminated`：线程的 `run()` 方法执行完毕，或者因为未捕获的异常而意外终止。

---

==补充：“特殊” 的中断方法==

线程中断方法 `interrupt()` 方法被调用时，会根据线程当前的状态有不同的表现；

当线程处于 `runnable` 或 `blocked` 状态时，只会将中断标志置为 `true`，线程状态本身**不会改变**，而是会继续运行或等待锁；

当线程处于 `waitting` 或者 `timed_waitting` 状态时调用中断方法，线程会被拉回 `runnable` 状态；

- 如果使用 `LockSupport.park()` 类方法进入 `waitting` 状态，线程会被正常唤醒，将中断标志位置为 `true`；
- 如果是调用其他方法进入 `waitting` 状态，则会抛出异常，同时**清空中断标志**；如果此时在 `catch` 块中不准备将异常继续抛出，最好手动调用 `Thread.currentThread().interrupt()` 方法设置中断标志位，交给上层方法处理。

---