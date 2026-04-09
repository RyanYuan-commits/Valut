---
type: permanent
banner: 附件/Banner/pexels-bertellifotografia-1144690.jpg
---
## 1	ByteBuf 的优势

与 `java.nio.Buffer` 相比，`ByteBuf` 有如下优势：

- 通过池化机制减少了内存的分配和释放次数；
- 使用零复制机制，减少了内存的复制；
- 不需要调用 `flip` 切换读写模式；
- 可拓展性好，可以自定义缓冲区类型；
- 读取和写入下标是分开的，更好理解；
- 支持方法的链式调用；
- 支持引用计数，方便重复使用。

## 2	组成部分

`ByteBuf` 使用内部的字节数组来缓冲数据，这个数组从逻辑上分为四个部分：

- ==已用字节==：已用完的、废弃的无效字节；
- ==可读字节==：`ByteBuf` 保存的有效数据，从 `ByteBuf` 中读取的数据都来自这一部分；
- ==可写字节==：写入到 `ByteBuf` 的数据都存入这一部分；
- ==可扩容字节==：`ByteBuf` 的最大可扩容量。

## 3	核心属性

```java
int readerIndex;
int writerIndex;
private int maxCapacity;
```

这些属性定义在 `AbstractByteBuf` 中，将这些属性标注在 `ByteBuf` 内部数组上大致为：

![[ByteBuf 的核心字段.png|800]]

`maxCapacity` 表示 `ByteBuf` 的最大容量，在写入数据时，如果容量不足，可以进行扩容。

## 4	核心方法

### 4.1	容量系列方法

`capacity()`：表示 `ByteBuf` 的容量，是已用、可读、可写三部分容量的总和；

`maxCapacity()`：返回当前 `ByteBuf` 的 `maxCapacity` 属性。

### 4.2	写入系列方法

