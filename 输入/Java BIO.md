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

---

Java IO 的命名极其规范，通常遵循这样的公式：

- **节点流**（直接干活的）：`数据源/目的地 + 基类名`（如 `File` + `InputStream`）
- **处理流**（包装增强的）：`特性/功能 + 基类名`（如 `Buffered` + `Reader`）

---

节点流：节点流是真正与底层（文件、内存）打交道的流。它们是最内层的“搬运工”。

| **数据源类型**          | **字节流 (InputStream/OutputStream)**                    | **字符流 (Reader/Writer)**                    | **适用场景与说明**                                          |
| ------------------ | ----------------------------------------------------- | ------------------------------------------ | ---------------------------------------------------- |
| **文件 (File)**      | `FileInputStream`<br><br>  <br><br>`FileOutputStream` | `FileReader`<br><br>`FileWriter`           | **最常用。** 用于读写本地磁盘上的文件。`FileReader` 内部其实默认封装了转换流。     |
| **内存数组 (Array)**   | `ByteArrayInputStream`<br><br>`ByteArrayOutputStream` | `CharArrayReader`<br><br>`CharArrayWriter` | 直接在内存中开辟数组进行数据周转，**不涉及磁盘 IO**。常用于网络数据包的拼接或小块数据的临时缓存。 |
| **内存字符串 (String)** | 无（字符串本质是字符，不适用于字节流）                                   | `StringReader`<br><br>`StringWriter`       | 专门用于在内存中处理字符串数据，常用于需要以字符流形式读取一段已知字符串的场景。             |

处理流/装饰流：处理流不能直接访问数据源，它们必须**包装**在一个已有的流（节点流或其他处理流）之上，为其提供额外的超能力。

| **增强功能**             | **字节流包装类**                                          | **字符流包装类**                                      | **核心作用与说明**                                                                                                          |
| -------------------- | --------------------------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **性能缓冲 (Buffering)** | `BufferedInputStream`<br><br>`BufferedOutputStream` | `BufferedReader`<br><br>`BufferedWriter`        | **极度常用。** 提供 8KB 的内存缓冲区。大大减少向操作系统发起底层 IO 调用的次数。`BufferedReader` 特有 `readLine()`。                                     |
| **字节转字符 (Bridging)** | 无                                                   | `InputStreamReader`<br><br>`OutputStreamWriter` | **核心桥梁。** BIO 体系中唯一能将字节流转化为字符流的类。在网络编程（获取的是字节）转文本处理时必用，且在此处指定字符集以解决乱码。                                               |
| **基本数据类型 (Data)**    | `DataInputStream`<br><br>`DataOutputStream`         | 无                                               | 允许你直接读写 Java 的基础数据类型（如 `writeInt()`, `readDouble()`），而不需要你自己去将 `int` 拆解为 4 个字节。                                      |
| **对象序列化 (Object)**   | `ObjectInputStream`<br><br>`ObjectOutputStream`     | 无                                               | **RPC 框架基础。** 将 Java 对象打碎成字节序列以便保存或网络传输（序列化），以及将字节拼装回对象（反序列化）。被操作的对象必须实现 `Serializable` 接口。                          |
| **打印输出 (Printing)**  | `PrintStream`                                       | `PrintWriter`                                   | 提供极其方便的 `print()` 和 `println()` 方法。我们每天用的 `System.out` 就是一个 `PrintStream`。在 Web 编程中，`PrintWriter` 常用于向客户端输出 HTML 响应。 |
