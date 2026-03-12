---
type: permanent
banner: Assets/Banner/Pasted image 20251122105012.png
aliases:
  - Synchronized
  - synchronized
---
# 🌐 核心观点

synchronized 是 Java 语言中的一个关键字, 用于实现线程同步, 确保多个线程在访问共享资源时的互斥性, 可见性和有序性. 它是一种内置的, 基于监视器锁的同步机制.. 它的实现方式是一个逐步优化的过程, 涉及字节码层面, JVM 运行时层面和机器码层面, 其核心思想是从高性能的乐观锁逐步升级到重量级的悲观锁.

---

# 🔖 详细解释

## 1	监视器锁

synchronized 是 Java 的一个关键字, 用于实现**线程同步**, 也就是解决原子性, 可见性和有序性的问题. 是一种内置的, 基于**监视器锁**的同步机制.

```java
public class Main { 
	static int a = 0; 
	public static void main(String[] args) throws InterruptedException { 
		synchronized (Main. class) { 
			a += 1; 
		} 
	} 
}

// ====== bytecode ======

 0 ldc #2 <com/ryan/Main>
 2 dup
 3 astore_1
 4 monitorenter
 5 getstatic #3 <com/ryan/Main. a : I>
 8 iconst_1
 9 iadd
10 putstatic #3 <com/ryan/Main. a : I>
13 aload_1
14 monitorexit
15 goto 23 (+8)
18 astore_2
19 aload_1
20 monitorexit
21 aload_2
22 athrow
23 return
```

当 JVM 遇到 synchronized 关键字时, 会在编译后的字节码中插入两条指令:

- monitorenter: 在进入同步块时尝试获取对象的监视器锁.
	
- monitorexit: 在退出同步块(包括正常退出和异常退出)时释放锁.

编译器会自动生成一个异常处理表, 确保无论临界区代码是正常结束还是异常退出, monitorexit 指令都会被执行, 从而避免死锁. 这就是为什么 synchronized 是隐式锁, 无需手动释放.



![[Java monitor 锁结构.png]]

上图是监视器锁的基本结构, 它由这三部分构成:

- Entry Set: 等待获取对象锁的线程.
	
- The Owner: 获取到锁的线程(只能有一个), 获取到锁的线程可以通过 notify 等方法来唤醒其他线程.
	
- Wait Set: 等待的线程, 比如在 synchronized 代码块中调用 wait 方法的线程.

## 2	对象头与锁升级

但是 monitor 依赖的是操作系统级别的 mutex lock, 使用它会导致用户态和内核态的切换, 带来很大的性能开销. 所以在 JDK6 之后, 引入了锁升级的机制, synchronized 是依赖对象头中的 Mark World 来实现的, 里面会含有对象的锁信息:

![[对象头中锁状态的存储.png|800]]

锁分为以下的四种状态:

- 无锁: 没有对资源进行锁定, 所有线程都可以对资源进行访问; 适用于这个变量不会出现在多线程的环境下, 或者即使出现在多线程的环境下, 但我不想对这个变量进行保护的情况, 或者使用无锁编程的方式来修改这个变量.
	
- 偏向锁: 一个对象被加锁了, 但是假设锁总是由一个线程获得, 此时就会采用偏向锁的方式去优化, 对象头中存储的是线程 ID, 当线程需要获取锁的时候, 检查线程的 ID 是否为对象头中存储的 ID, 如果是的话, 就可以直接让线程获取到锁.
	
- 轻量级锁: 当对象处于偏向锁的状态, 而此时有另外的线程进行竞争的话, 轻量级锁会升级为偏向锁, 此时对象头中存储的是指向栈中锁的记录的指针. 当线程通过 CAS 获取到锁之后, 会存储一份线程对象头的副本到自己的方法栈中的 Lock Record 中, 同时, 这个区域中会有一个 owner 指针指向对象.对象中又存有一个指向 Lock Record 中锁信息的指针, 这样, 锁和线程都知道了对方的存在;此时再有其他线程希望获取这个对象锁的时候, 就会轮询去尝试获取锁. 自旋操作相比于 CPU 调度, 在能较快获取到锁的场景下会有极好的表现.
	
- 重量级锁: 一旦等待的线程超过一个, 轻量级锁会被升级为重量级锁, 它的实现就是我们前面提到的 mutex lock, 是依赖操作系统实现的监视器锁.


---

# 📚 参考内容

