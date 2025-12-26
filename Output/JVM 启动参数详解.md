---
type: permanent
---
# 🌐 核心观点

用一两句话概括这个原子化的想法.

---

# 🔖 详细解释

## 1	分类

JVM 的启动参数, 从形式上可以简单分为:

- 以 - 开头的标准参数, 所有的 JVM 实现都要支持这些参数, 并且向后兼容;
	
- 以 -X 开头的非标准参数, 默认的 JVM 实现支持这些参数, 但并不保证所有 JVM 都支持, 并且不保证向后兼容;
	
- 以 -XX 开头的非稳定参数, 专门用于控制 JVM 的行为, 跟具体的 JVM 实现有关, 可能会随版本更新取消;

通过 -XX:(+/-)Flags 的形式开关某个功能, 或者通过 -XX:key=value 的方式配置某个参数的值.

## 2	设置系统属性

给 Java 程序传递参数, 常用的方法有:

- 系统属性(环境变量): 通过 -Dkey=value 的方式直接传递, 比系统环境变量中的参数优先级高;
	
- 命令行参数: 在启动命令后传递, 可以在 args 数组获取到.

系统属性通过 System.getProperty("key") 的方式获取, 常见的设置有:

- -Duser.timezone=GMT+08: 设置用户的时区为东八区;
	
- -Dfile.encoding=UTF-8: 设置默认的文件编码为 UTF-8.

## 3	Agent 相关选项

- -agentlib:libname 启用 native 方式的 agent, 参考 LD_LIBRARY_PATH 路径.
	
- -agentpath:pathname 启用 native 方式的 agent.
	
- -javaagent:jarpath 启用外部的 agent 库, 比如 pinpoint. jar 等等.
	
- -Xnoagent 则是禁用所有 agent.

以下示例开启 CPU 使用时间抽样分析:

```java
JAVA_OPTS="-agentlib:hprof=cpu=samples, file=cpu. samples. log"
```

其中 hprof 是 JDK 内置的一个性能分析器. cpu=samples 会抽样在各个方法消耗的时间占比, Java 进程退出后会将分析结果输出到文件.

## 4	JVM 运行模式

JVM 有两种运行模式:

- -server: 设置 jvm 使 server 模式, 特点是启动速度比较慢, 但运行时性能和内存管理效率很高, 适用于生产环境. 在具有 64 位能力的 jdk 环境下将默认启用该模式, 而忽略 -client 参数.
	
- -client: JDK1. 7 之前在 32 位的 x86 机器上的默认值是 -client 选项. 设置 jvm 使用 client 模式, 特点是启动速度比较快, 但运行时性能和内存管理效率不高, 通常用于客户端应用程序或者 PC 应用开发和调试.

JVM 加载字节码后, 可以解释执行或编译成本地代码再执行, 所以可以配置 JVM 对字节码的处理模式:

- -Xint:在解释模式(interpreted mode)下, -Xint 标记会强制 JVM 解释执行所有的字节码, 这当然会降低运行速度, 通常低 10 倍或更多.
	
- -Xcomp:-Xcomp 参数与 -Xint 正好相反, JVM 在第一次使用时会把所有的字节码编译成本地代码, 从而带来最大程度的优化.
	
- -Xmixed:-Xmixed 是混合模式, 将解释模式和变异模式进行混合使用, 有 JVM 自己决定, 这是 JVM 的默认模式, 也是推荐模式. 我们使用 java -version 可以看到 mixed mode 等信息.

## 5	设置堆内存

内存设置是 GC 分析和调优的重点.

- -Xmx: 指定最大堆内存. 如 -Xmx4g. 这只是限制了 Heap 部分的最大值为 4g. 这个内存**不包括栈内存**, 也**不包括堆外使用的内存**.

- -Xms: 指定堆内存空间的初始大小. 如 -Xms4g. 而且指定的内存大小, 并不是操作系统实际分配的初始值, 而是 GC 先规划好, 用到才分配. **专用服务器**上需要保持 -Xms 和 -Xmx **一致**, 否则应用刚启动可能就有好几个 FullGC. 当两者配置不一致时, 堆内存扩容可能会导致性能抖动.
	
- -Xmn: 等价于 -XX:NewSize, 使用 G1 垃圾收集器**不应该**设置该选项, 在其他的某些业务场景下可以设置. 官方建议设置为 **-Xmx 的 1/2 ~ 1/4**.
	
- -XX:MaxPermSize=size: 这是 JDK1. 7 之前使用的. Java8 默认允许的 Meta 空间无限大, 此参数无效.

- -XX:MaxMetaspaceSize=size: Java8 默认不限制 Meta 空间, 一般不允许设置该选项.

- -XX:MaxDirectMemorySize=size: 系统可以使用的最大**堆外内存**, 这个参数跟 -Dsun.nio.MaxDirectMemorySize 效果相同.

- -Xss: 设置**每个线程栈的字节数**. 例如 -Xss1m 指定线程栈为 1MB, 与-XX:ThreadStackSize=1m 等价

### 5.1	堆外内存

一个 Java 进程里面, 可以分配 native memory 的东西有很多, 特别是使用第三方 native 库的程序更是如此.

但在这里面除了

- GC heap = Java heap + Perm Gen (JDK <= 7)
	
- Java thread stack = Java thread count * Xss
	
- other thread stack = other thread count * stack size
	
- CodeCache 等东西之外

还有诸如 HotSpot VM 自己的 StringTable, SymbolTable, SystemDictionary, CardTable, HandleArea, JNIHandleBlock 等许多数据结构是常驻内存的, 外加诸如 JIT 编译器, GC 等在工作的时候都会额外临时分配一些 native memory, 这些都是 HotSpot VM 自己所分配的 native memory;在 JDK 类库实现中也有可能有些功能分配长期存活或者临时的 native memory.

然后就是各种第三方库的 native 部分分配的 native memory.

"Direct Memory", 一般来说是 Java NIO 使用的 Direct-X-Buffer(例如 DirectByteBuffer)所分配的 native memory, 这个地方如果我们使用 netty 之类的框架, 会产生大量的堆外内存.

### 5.2	最佳实践

- -Xmx 不要设置的和系统内存一致, 推荐配置系统或容器里可用内存的 70-80% 最好. 比如说系统有 8G 物理内存, 系统自己可能会用掉一点, 大概还有 7.5G 可以用, 那么建议配置 7.5 * 0.8 = 6G;
	
- 专用服务器的 -Xmx 和 -Xms 建议设置成一样的. 

## 6	GC 日志相关的参数

- -verbose:gc :和其他 GC 参数组合使用, 在 GC 日志中输出详细的 GC 信息. 包括每次 GC 前后各个内存池的大小, 堆内存的大小, 提升到老年代的大小, 以及消耗的时间. 此参数支持在运行过程中动态开关. 比如使用 jcmd, jinfo, 以及使用 JMX 技术的其他客户端.
	
- -XX:+PrintGCDetails 和 -XX:+PrintGCTimeStamps:打印 GC 细节与发生时间. 请关注我们后续的 GC 课程章节.
	
- -Xloggc:file: 与-verbose:gc 功能类似, 只是将每次 GC 事件的相关情况记录到一个文件中, 文件的位置最好在本地, 以避免网络的潜在问题. 若与 verbose:gc 命令同时出现在命令行中, 则以 -Xloggc 为准.

## 7	指定垃圾收集器相关参数

- -XX:+UseG1GC:使用 G1 垃圾回收器
	
- -XX:+UseConcMarkSweepGC:使用 CMS 垃圾回收器
	
- -XX:+UseSerialGC:使用串行垃圾回收器
	
- -XX:+UseParallelGC:使用并行垃圾回收器

## 8	特殊情况执行脚本的参数

- -XX:+-HeapDumpOnOutOfMemoryError: 当 **OutOfMemoryError** 产生, 自动 Dump 堆内存. 因为在运行时并没有什么开销, 所以在生产机器上是可以使用的. 
	
- -XX:HeapDumpPath 选项, 与 HeapDumpOnOutOfMemoryError 搭配使用, 指定内存溢出时 Dump 文件的目录. 如果没有指定则默认为启动 Java 程序的工作目录. 示例用法: java -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/usr/local/ ConsumeHeap 自动 Dump 的 hprof 文件会存储到 /usr/local/ 目录下.
	
- -XX:OnError 选项, 发生致命错误时(fatal error)执行的脚本. 例如, 写一个脚本来记录出错时间, 执行一些命令, 或者 curl 一下某个在线报警的 url. 示例用法: java -XX:OnError="gdb - %p" MyApp 可以发现有一个 %p 的格式化字符串, 表示进程 PID.
	
- -XX:OnOutOfMemoryError 选项, 抛出 OutOfMemoryError 错误时执行的脚本.
	
- -XX:ErrorFile=filename 选项, 致命错误的日志文件名, 绝对路径或者相对路径.


---

# 📚 参考内容

- [jvm 启动参数详解: 博观而约取, 厚积而薄发](https://o.alldu.cn/docs/jvm-%E6%A0%B8%E5%BF%83%E6%8A%80%E6%9C%AF-32-%E8%AE%B2%E5%AE%8C/08-jvm-%E5%90%AF%E5%8A%A8%E5%8F%82%E6%95%B0%E8%AF%A6%E8%A7%A3%E5%8D%9A%E8%A7%82%E8%80%8C%E7%BA%A6%E5%8F%96%E5%8E%9A%E7%A7%AF%E8%80%8C%E8%96%84%E5%8F%91/)
	
- [Java HotSpot VM Options](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)