---
type: permanent
banner: 附件/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
## 1	基本介绍

nio 提供 buffer 组件以实现**非阻塞读写**，channel 读写的数据都必须经过 buffer，read 操作将数据从 channel 读取到缓冲区，write 操作将数据从缓冲区写入到 channel 中；buffer 是面向流读写的 old io 所没有的，也是 nio 非阻塞的前提和基础之一。

![[channel 的读写都需要经过 buffer.png|900]]

`Buffer` 是一个抽象类，对应 java 的主要数据类型，nio 提供了 `ByteBuffer`、`CharBuffer`、`DoubleBuffer`、`FloatBuffer`、`IntBuffer`、`LongBuffer`、`ShortBuffer`、`MappedByteBuffer` 八中的实现；其中 `MappedByteBuffer` 比较特殊，专门用于内存映射；比如，在零拷贝的 mmap 实现中，用于表示被映射的那块内核缓冲区的内存。

## 2	核心属性

每个 buffer 都会对应一块内存，buffer 的缓冲功能通过操控这块内存来实现，在 `ByteBuffer` 中，这块内存被定义为 `final byte[] hb`，`Buffer` 类提供了 `capacity`、`position`、`limit` 几个属性来描述这块内存的使用状态。

- ==容量（capacity）==：`capacity` 表示内部容量的大小，写入的数据量超过 `capacity`，代表缓冲区已满，无法继续写入；`Buffer` 类实例在初始化时候，会按照 `capacity` 分配内部数据的内存，所以 `capacity` 在设置后，就**无法被修改**。
- ==下一个可读或可写的位置（position）==：当完成一个单位的读取或写入后，`position` 会移动到**下一个**可读或可写的位置；
- ==可读或可写的上限（limit）==：表示当前可读或可写的最大上限。

## 3	Buffer 类的重要方法

### 3.1	缓冲区创建

```java
// position = 0; mark = -1; limit = 20; capacity = 20
IntBuffer buffer = IntBuffer.allocate(20); 
```

`allocate` 是 `Buffer` 子类的静态方法，入参一般是容量，用于创建对应的 `Buffer` 实例；`Buffer` 实例在创建后，处于写入模式。

### 3.2	写入缓冲区

```java
IntBuffer intBuffer = IntBuffer.allocate(20);  
  
for (int i = 0; i < 5; i++) {  
    intBuffer.put(i);
}

// capacity = 20; position = 5; limit = 20; mark = -1
```

### 3.3	切换读取和写入模式

```java
public Buffer flip() {  
    limit = position;  
    position = 0;  
    mark = -1;  
    return this;
}
```

通过 `flip()` 方法，可以让缓冲区在读取和写入模式之间转换；

当从写模式转到读模式时，可读的最大值为当前写入的位置（`limit = po），