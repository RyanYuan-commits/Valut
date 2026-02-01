---
type: permanent
---
# 🌐 关键词

Java, Collection, ArrayList

---

# 🔖 详细解释

## 1	无参构造

```java
public ArrayList() {
	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

将 `elementData` 赋值为 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`, 这是个空数组. 除此之外, 还有一个名为 `EMPTY_ELEMENTDATA` 的空数组, 当指定初始容量为 0 的时候, `elementData` 会被赋值为 `EMPTY_ELEMENTDATA`, biao's.

## 2	指定初始容量构造

```java
 public ArrayList(int initialCapacity) {
 if (initialCapacity > 0) {
 this. elementData = new Object[initialCapacity];
 } else if (initialCapacity == 0) {
 this. elementData = EMPTY_ELEMENTDATA;
 } else {
 throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
 }
 }
 
 // 无参构造方法
 public ArrayList() {
 this. elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
 }

```
- 如果指定的初始容量大于零, 就直接构造一个容量为初始容量的 Object 数组, 然后将其赋值给 elementData;
- 否则, 当初始的容量等于 0 的时候, 将 elementData 赋值为 EMPTY_ELEMENTDATA;
- 如果是负数, 抛异常.
## 3	基于其他 List 构造

```java
public ArrayList(Collection<? extends E> c) {
 Object[] a = c. toArray();
 if ((size = a. length) != 0) {
 if (c. getClass() == ArrayList. class) {
 elementData = a;
 } else {
 elementData = Arrays. copyOf(a, size, Object[]. class);
 }
 } else {
 // replace with empty array.
 elementData = EMPTY_ELEMENTDATA;
 }
}
```
- 首先通过 Collction 接口中定义的 toArray() 方法, 将集合类转为一个数组, 方便后续的处理;
- 然后在 if 的条件中去指定 size(当前 ArrayList 实例中存放元素的个数)为数组的长度, 然后判断这个长度是否为 0;
- 如果 c. getClass() 为 ArrayList 的话, 就直接将数组 a 赋值给 elementData , 如果不是的话, 需要再做一次转化, 通过 Arrays. copyOf() 方法将数组复制为对象数组, 这样做的原因有两个:
	- 一是为了保证类型安全;
	- 二是为了防止那个自定义的集合将把自己内部的数组的引用传过来, 此时两个 List 底层操控的是相同的数组, 会引起很多意料之外的问题.