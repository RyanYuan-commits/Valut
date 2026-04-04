---
source:
type: input
banner: 附件/Banner/pexels-8kspain-21564213.jpg
---
BIO 是同步阻塞模型，在数据拷贝阶段，需要应用程序主动发起；在数据准备阶段，线程会挂起等待；

---

Java IO 采用了装饰器模式，按照数据流的角色分为节点流和装饰流；

- 节点流负责直接从数据源读写，如 `FileInputStream`；
- 装饰流则在**节点流的基础上**包装，提供增强功能，如 `BufferedInputStream` 提供缓冲功能。

而根据操控的数据单位，数据流又可以分为字节流和字符流：

- 字节流处理的是 8-bit 的原始字节，基类是 `InputStream` 和 `OutputStream`，适合处理图片、视频等二进制文件；
- 字符流处理的是 16-bit 的 Unicode 字符，基类是 `Reader` 和 `Writer`，适合处理纯文本。

