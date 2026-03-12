---
type: permanent
banner: Assets/Banner/pexels-vadimsadovski-33522604-7049988.jpg
---
---

**关键词**: java, strong reference, soft reference, weak reference, phantom reference

---

## 1	介绍

**强引用** (StrongReference): 创建一个对象并把这个对象赋给一个引用变量, 如果内存不足, JVM 会抛出 OOM 错误也不会回收对象,  想中断强引用和某个对象之间的关联, 可以显示地将引用赋值为 null. 

**软引用** (SoftReference): 如果一个对象具有软引用, 内存空间足够, 垃圾回收器就不会回收它；**如果内存空间不足了, 就会回收这些对象的内存**. 软引用可用来实现内存敏感的高速缓存,比如网页缓存、图片缓存等, 使用软引用能防止内存泄露, 增强程序的健壮性. Java 中可以通过 `SoftReference` 类来实现软引用.

**弱引用** (WeakReference): 当 JVM 进行垃圾回收时, **无论内存是否充足**, 都会回收被弱引用关联的对象, 弱引用通常是用来描述非必需对象. 

**虚引用** (PhantomReference): 与其他引用类型不同, `PhantomReference` 的 `get()` 方法总是返回 `null`. 因此, 无法通过 `PhantomReference` 访问其引用的对象. 它的主要用途是跟踪对象被垃圾回收的过程, 通常与 `ReferenceQueue` 一起使用. 当垃圾回收器准备回收一个对象时, 如果发现它有一个 `PhantomReference`, 就会在回收对象之前, 将这个引用加入到与之关联的 `ReferenceQueue` 中. 

## 2	代码示例

```java
// 对 "new Object()" 的强引用  
Object strongReference = new Object();

// 对 "new Object()" 的弱引用  
Object strongReference = new Object();  
WeakReference<Object> weakReference = new WeakReference<>(strongReference);  
Object weakReferenceObject = weakReference.get();

// 对 "new Object()" 的软引用  
Object strongReference = new Object();  
SoftReference<Object> softReference = new SoftReference<>(strongReference);  
Object softReferenceObject = softReference.get();

// 使用虚引用监控对象状态
Object object = new Object();  
ReferenceQueue<Object> objectReferenceQueue = new ReferenceQueue<>();  
new PhantomReference<>(object, objectReferenceQueue);  
  
if (objectReferenceQueue.poll() != null) {  
    System.out.println("对象被回收了");  
}
```

## 3	引用处理速度优化

对于特殊的引用, 垃圾回收器不能简单的回收它们, 需要根据它们的类型进行特殊的处理, 这个过程在 GC 日志中的关键字为 "Reference Processing".

G1 收集器默认希望利用多核 CPU 来加速引用处理, 它使用了一种启发式的策略来决定启动多少线程, 每当处理一定数量的引用就多启动一个线程, 这个数量通过 JVM 参数 `-XX:ReferencesPerThread` 控制, 最大值受 `-XX:ParallelGCThreads` 制约, 这个启发式的策略可以通过如下的方式调整:

- 设置 `-XX:ReferencesPerThread=0` 来禁用该规则, 默认使用所有可用线程;
	
- 设置 `-XX:-ParallelRefProcEnabled` 来完全禁用并行化.

当发现这个阶段花费时间较长时, 可以通过 `-XX:ReferencesPerThread=0` 来优化.

---