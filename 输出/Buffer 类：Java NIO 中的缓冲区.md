---
type: permanent
banner: 附件/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
## 1	基本介绍

nio 提供 buffer 组件以实现**非阻塞读写**，channel 读写的数据都必须经过 buffer，read 操作将数据从 channel 读取到缓冲区，write 操作将数据从缓冲区写入到 channel 中；buffer 是面向流读写的 old io 所没有的，也是 nio 非阻塞的前提和基础之一。

![[channel 的读写都需要经过 buffer.png|900]]

`Buffer` 是一个抽象类，对应 java 的主要数据类型，nio 提供了 `ByteBuffer`、`CharBuffer`、`DoubleBuffer`、`FloatBuffer`、`IntBuffer`、`LongBuffer`、`ShortBuffer`、`MappedByteBuffer` 八中的实现；其中 `MappedByteBuffer` 比较特殊，专门用于内存映射；比如，在零拷贝的 mmap 实现中，用于表示被映射的那块内核缓冲区的内存。

## 2	核心属性

每个 buffer 都会对应一块内存，buffer 的缓冲功能通过操控这块内存来实现，在 `ByteBuffer` 中，这块内存被定义为 `final byte[] hb`，`Buffer` 类提供了 `capacity`、`position`、`limit` 和 `mark` 几个属性来描述这块内存的使用状态。

- ==容量（capacity）==：`capacity` 表示内部容量的大小，写入的数据量超过 `capacity`，代表缓冲区已满，无法继续写入；`Buffer` 类实例在初始化时候，会按照 `capacity` 分配内部数据的内存，所以 `capacity` 在设置后，就**无法被修改**。
- 




