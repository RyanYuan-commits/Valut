---
type: permanent
banner: Assets/Banner/pexels-8kspain-21564213.jpg
aliases:
  - jmp
---
# 🌐 核心观点

jmap, 全称 Java Memory Map, 是一个内置的命令行工具, 用于生成 Java 堆的快照.

---

# 🔖 详细解释

## 1	基本介绍

jmap, 全称 Java Memory Map, 是一个内置的**命令行工具**, 用于**生成 Java 堆的快照**, 用来查看堆内存的详细信息, 如:

- 各个内存区域的使用情况;
	
- 内存中的对象统计信息, 如实例数量, 占用空间;
	
- 类加载信息, finalizer 队列等.

是一个 "**事后**" 分析工具, 通常在应用出现内存泄漏, 内存溢出时使用.

## 2	语法

```bash
jmap [option] <pid>        # 连接到正在运行的Java进程(通过进程ID PID)
jmap [option] <executable <core>   # 连接到一个核心转储文件
jmap [option] [server_id@]<remote server IP or hostname> # 连接到远程调试服务器
```

pid 是 Java 进程的 ID, 可以使用 [[Java 进程状态查看工具 - jps|jps]] 命令来查看当前机器上所有当前用户可见的 Java 进程 ID.

## 3	常用参数

### 3.1	-dump:\[live,]format=b,file=<file_name>

`-dump:[live,]format=b,file=<file_name>` 是最常用的选项:

- `live`: 是一个可选参数, 代表只转储**存活对象**, 会触发一次 **Full GC**, 但得到的转储文件**更小**, 更适合分析**内存泄露**;
	
- `format=b`: 指定转储格式为二进制, 目前是唯一支持的格式;
	
- `file=<file_name>`: 指定输出文件的名称, 如 heapdump.bin.

生成的 .bin 文件可以使用更专业的分析工具, 如 Eclipse MAT, JProfiler, VisualVM 进行加载和分析.

### 3.2	-heap

`-heap` 用于显示 Java 堆的详细信息, 输出的内容包括:

- 使用的垃圾收集器类型;
	
- 堆的配置参数;
	
- 各内存区域的当前使用情况.

```java
// Example: Java Heap Information from jmap -heap Command
$ jmap -heap 29620
Attaching to process ID 29620, please wait...
Debugger attached successfully.
Client compiler detected.
JVM version is 1.6.0-rc-b100

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
   MinHeapFreeRatio = 40
   MaxHeapFreeRatio = 70
   MaxHeapSize      = 67108864 (64.0MB)
   NewSize          = 2228224 (2.125MB)
   MaxNewSize       = 4294901760 (4095.9375MB)
   OldSize          = 4194304 (4.0MB)
   NewRatio         = 8
   SurvivorRatio    = 8
   PermSize         = 12582912 (12.0MB)
   MaxPermSize      = 67108864 (64.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 2031616 (1.9375MB)
   used     = 70984 (0.06769561767578125MB)
   free     = 1960632 (1.8698043823242188MB)
   3.4939673639112905% used
Eden Space:
   capacity = 1835008 (1.75MB)
   used     = 36152 (0.03447723388671875MB)
   free     = 1798856 (1.7155227661132812MB)
   1.9701276506696428% used
From Space:
   capacity = 196608 (0.1875MB)
   used     = 34832 (0.0332183837890625MB)
   free     = 161776 (0.1542816162109375MB)
   17.716471354166668% used
To Space:
   capacity = 196608 (0.1875MB)
   used     = 0 (0.0MB)
   free     = 196608 (0.1875MB)
   0.0% used
tenured generation:
   capacity = 15966208 (15.2265625MB)
   used     = 9577760 (9.134063720703125MB)
   free     = 6388448 (6.092498779296875MB)
   59.98769400974859% used
Perm Generation:
   capacity = 12582912 (12.0MB)
   used     = 1469408 (1.401336669921875MB)
   free     = 11113504 (10.598663330078125MB)
   11.677805582682291% used
```

### 3.3	-histo:\[live]

显示堆中**对象**的统计信息, 包括**类名**, **实例数量**, **合计容量**, -live 参数用于只统计存活对象, 这个命令执行的**非常快**, 是快速查看**哪个类占用了最多内存**的首选方法.

```java
// Example: Class Specific Histogram of Java Heap for a Process
$ jmap -histo 29620
num   #instances    #bytes  class name
--------------------------------------
  1:      1414     6013016  [I
  2:       793      482888  [B
  3:      2502      334928  <constMethodKlass>
  4:       280      274976  <instanceKlassKlass>
  5:       324      227152  [D
  6:      2502      200896  <methodKlass>
  7:      2094      187496  [C
  8:       280      172248  <constantPoolKlass>
  9:      3767      139000  [Ljava.lang.Object;
 10:       260      122416  <constantPoolCacheKlass>
 11:      3304      112864  <symbolKlass>
 12:       160       72960  java2d.Tools$3
 13:       192       61440  <objArrayKlassKlass>
 14:       219       55640  [F
 15:      2114       50736  java.lang.String
 16:      2079       49896  java.util.HashMap$Entry
 17:       528       48344  [S
 18:      1940       46560  java.util.Hashtable$Entry
 19:       481       46176  java.lang.Class
 20:        92       43424  javax.swing.plaf.metal.MetalScrollButton
... more lines removed here to reduce output...
1118:         1           8  java.util.Hashtable$EmptyIterator
1119:         1           8  sun.java2d.pipe.SolidTextRenderer
Total    61297    10152040
```

### 3.4	-clstats

`-clstats` 参数以**类加载器**为维度, 显示类加载器的**统计信息**, 以及它们加载的类的**元数据**占用的**内存量**, 用于诊断类加载器泄露或元空间问题.

- 类加载器的地址;
	
- 被加载的类的数量;
	
- 当前类加载器加载到元空间的数据耗费的大概字节数;
	
- 父类加载器;
	
- 类加载器对象是否会被垃圾回收;
	
- 类加载器的类名.

## 4	可视化分析工具

### 4.1	Memory Analyzer

[Memory Analyzer (MAT)](https://eclipse.dev/mat/), 是 Eclipase 基金会的开源项可以分析包含数亿级别对象的堆, 快速计算每个对象占用内存大小, 以及对象之间的引用关系, 自动检测内存泄漏的嫌疑对象等.

![[MAT 界面.png|800]]

---

# 📚 参考内容

- [The jmp Utility](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr014.html#BABJIIIH)