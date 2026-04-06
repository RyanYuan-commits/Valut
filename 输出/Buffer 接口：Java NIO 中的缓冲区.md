---
type: permanent
banner: 附件/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
## 1	基本介绍

为了实现非阻塞的读写，NIO 提供了 buffer 组件，channel 读写的数据都必须经过 buffer，read 操作将数据从 channel 读取到缓冲区，write 操作将数据从缓冲区写入到 channel 中；

![[channel 的读写都需要经过 buffer.png|900]]

buffer 是面向流读写的 old io 所没有的，也是 nio 非阻塞的