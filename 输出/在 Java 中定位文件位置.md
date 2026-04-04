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

在 Intellij IDEA 或者 Eclipse 中，`user.dir` 是当前 project 或者 module 的根目录。

## 2	基于 classpath 的定位方式

### 2.1	通过 Class 加载资源

对应两个方法，`Class#getResource` 和 `Class#getResourceAsStream`，其路径解析规则为：

- 若传入的路径不以 `/` 开头，java 会以当前类所在的包路径作为起点去寻找文件；
  在 `com.example.App` 类中调用 `App.class.getResourceAsStream("config.txt")`，java 会尝试寻找 `com/example/config.txt` 文件；
- 若传入的路径以 `/` 开头，java 会在整个项目的根目录开始寻找；
  上面案例中，若路径为 `"/config.txt"`，会在 classpath 根目录寻找。

### 2.2	通过类加载器加载资源

对应方法为 `ClassLoader#getResource` 或 `ClassLoader#getResourceAsStream`；

类加载器寻找的起点永远是 classpath 的根目录，与 Class 传入路径以 `/` 时的表现一致，Class 只是去掉了 `/` 然后将剩余的加载部分交给类加载器完成。