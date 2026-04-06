---
type: permanent
banner: 附件/Banner/pexels-8kspain-21564213.jpg
---
通过调用 `FileChannel#map()` 方法获取 `MappedByteBuffer` 实例，系统会将文件直接映射到虚拟内存，java 程序操作这个 buffer 实质上相当于在操控操作系统的那块内存；对应零拷贝的 mmap 技术；

通过调用 `FileChannel#transferTo()` 和 `FileChannel#transferFrom()` 方法，底层会直接调用操作系统的 `sendfile` 指令，实现数据在内核管道直接的直接传输，完全避开了堆内存。

