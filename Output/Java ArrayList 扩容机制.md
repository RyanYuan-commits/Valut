---
type: permanent
banner: Assets/Banner/pexels-walidphotoz-1509582.jpg
---
# 🌐 关键词

Java, Collection, ArrayList

---

# 🔖 详细解释

## 1	扩容时机

当 ArrayList 执行 add 类操作的时候, 会调用 `ensureCapacityInternal()` 方法来确保存储空间足够:

```java
private void ensureCapacityInternal(int minCapacity) {
	ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
```

首先会调用 `calculateCapacity()` 方法来获取最小容量:

```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {
	if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
		return Math.max(DEFAULT_CAPACITY, minCapacity);
	}
	return minCapacity;
}
```

if 块用于处理初始化场景, `DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 标识该 `ArrayList` 实例是通过默认构造方法创建的.

随后调用 `ensureExplicitCapacity()` 执行扩容:

```java
private void ensureExplicitCapacity(int minCapacity) {
	modCount++;
	// overflow-conscious code
	if (minCapacity - elementData.length > 0) grow(minCapacity);
}
```

最终会调用到 `grow()` 方法:

```java
private void grow(int minCapacity) {
	// overflow-conscious code
	int oldCapacity = elementData.length;
	int newCapacity = oldCapacity + (oldCapacity >> 1);
	if (newCapacity - minCapacity < 0)
		newCapacity = minCapacity;
	if (newCapacity - MAX_ARRAY_SIZE > 0)
		newCapacity = hugeCapacity(minCapacity);
	// minCapacity is usually close to size, so this is a win:
	elementData = Arrays.copyOf(elementData, newCapacity);
}
```

看将当前容量扩展 1.5 倍后能否满足 `minCapacity`，如果不能则本次扩容为 `minCapacity`, 随后调用 `Arrays.copyOf(elementData, newCapacity)`; 将原本 `elementData` 中的内容移动到新拓展的长度为 `newCapacity` 的数组中，这就完成了一个完整的扩容；