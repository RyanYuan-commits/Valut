---
type: permanent
banner: 附件/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
**使用场景**：当类中含有类似 `AtomicInteger` 这样的原子字段，且这个类需要大量被创建时，可以通过 `AtomicIntegerFieldUpdater` 来将 `AtomicInteger` 字段降级为普通的 `int` 字段，来降低存储对象头带来的消耗；

```java
public class MyService {
    // 1. 定义普通 volatile 变量
    private volatile int status = 0;

    // 2. 创建静态更新器（全局共享一个即可）
    private static final AtomicIntegerFieldUpdater<MyService> STATUS_UPDATER =
        AtomicIntegerFieldUpdater.newUpdater(MyService.class, "status");

    public void start() {
        // 3. 原子性地将 0 修改为 1
        if (STATUS_UPDATER.compareAndSet(this, 0, 1)) {
            System.out.println("Start success!");
        } else {
            System.out.println("Already started.");
        }
    }
}
```

一般将 updater 定义为 `private static final` 对象，避免 updater 在外部定义，因为权限不足而导致异常；

定义的字段类型使用 `volatile` 修饰，确保可见性，且需要为 `int`，不能被 `final` 修饰。