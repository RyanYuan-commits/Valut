---
type: permanent
banner: 附件/Banner/pexels-bertellifotografia-1144690.jpg
---
## 1	基于物理文件系统的定位

最基础的定位方式，直接与操作系统的文件系统交互，主要依赖于 `java.io.File` 或 Java7 引入的更现代的 `java.nio.file.Paths`；

### 1.1	使用绝对路径定位

```java
Path path = Paths.get("C:", "data", "config.properties");
```

推荐使用 `java.nio.file.Paths#get` 方法，它能够自动处理不同系统的文件分割符。

### 1.2	使用相对路径定位

当使用类似 `new File("config.properties")` 的方式定位文件时，Java 会使用**当前的工作目录**作为基准路径来解析这个相对路径；
这个值可以通过 `System.getProperty("user.dir")` 获取。

