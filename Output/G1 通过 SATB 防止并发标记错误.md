---
type: permanent
banner: Assets/Banner/pexels-walidphotoz-1509582.jpg
---
---

**关键词**: G1, SATB, [[三色标记法]]

---

三色标记法当赋值器插入了一条或多条从黑色对象到白色对象的新引用且赋值器删除了全部从灰色对象到该白色对象的直接或间接引用 时会错误的将这个新对象标记为垃圾对象, SATB (Snapshot-At-The-Beginning, 原始快照) 破坏的是第二个条件, 可以看作当前的清除是在清除开始时构建的快照上进行的, 即使灰色对象删除了到白色对象的引用, 这些白色对象最终也会被标记为可达.

具体的实现方式是当一个对象的引用被替换时, 通过写屏障将旧引用记录下来, 在清除结束后以这些引用为根再次扫描.

```cpp
// share/vm/gc_implementation/g1/g1SATBCardTableModRefBS.hpp
// 这表明我们不需要访问任何 BarrierSet 数据结构，
// 因此可以从静态上下文中调用此函数。
template <class T> static void write_ref_field_pre_static(T* field, oop newVal) {
  // 从字段中加载堆对象指针
  T heap_oop = oopDesc::load_heap_oop(field);
  
  // 如果字段原本存储的不是空指针（即存在旧值）
  if (!oopDesc::is_null(heap_oop)) {
    // 将旧值对象加入 SATB 队列
    enqueue(oopDesc::decode_heap_oop(heap_oop));
  }
}

// share/vm/gc_implementation/g1/g1SATBCardTableModRefBS.cpp
void G1SATBCardTableModRefBS::enqueue(oop pre_val) {
  // 空指针应该已经被过滤掉了
  assert(pre_val->is_oop(true), "Error");
  
  // 如果 SATB 标记队列未激活，则直接返回
  if (!JavaThread::satb_mark_queue_set().is_active()) return;
  
  // 获取当前线程
  Thread* thr = Thread::current();
  
  // 如果是 Java 线程
  if (thr->is_Java_thread()) {
    JavaThread* jt = (JavaThread*)thr;
    // 将对象加入该线程的 SATB 队列
    jt->satb_mark_queue().enqueue(pre_val);
  } else {
    // 如果是非 Java 线程（如 VM 线程、GC 线程等）
    // 需要获取共享锁以保证线程安全
    MutexLockerEx x(Shared_SATB_Q_lock, Mutex::_no_safepoint_check_flag);
    // 将对象加入共享 SATB 队列
    JavaThread::satb_mark_queue_set().shared_satb_queue()->enqueue(pre_val);
  }
}
```

如果放入队列的对象本来就是要被清除的, 就会让它躲过这次 GC, 产生浮动垃圾.

---