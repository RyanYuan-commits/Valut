---
type: permanent
---
# 🌐 核心观点

讲解 JDK, JRE, JVM 之间的关系与 JDK 的发展历史

---

# 🔖 详细解释

## 1	JDK, JRE 与 JVM

### 1.1	概念辨析

JDK, Java Development Kit 是用于开发 Java 应用程序的软件开发工具集合, 包括了 Java 运行时的环境(JRE), 解释器(Java), 编译器(javac), Java 归档(jar), 文档生成器(Javadoc)等工具. 简单的说我们要开发 Java 程序, 就需要安装某个版本的 JDK 工具包.

JRE, Java Runtime Enviroment 提供 Java 应用程序执行时所需的环境, 由 Java 虚拟机(JVM), 核心类, 支持文件等组成. 简单的说, 我们要是想在某个机器上运行 Java 程序, 可以安装 JDK, 也可以只安装 JRE, 后者体积比较小.

JVM, Java Virtual Machine 有三层含义, 分别是:

- JVM 规范要求;
	
- 满足 JVM 规范要求的一种具体实现(一种计算机程序);
	
- 一个 JVM 运行实例, 在命令提示符下编写 Java 命令以运行 Java 类时, 都会创建一个 JVM 实例, 我们下面如果只记到 JVM 则指的是这个含义;如果我们带上了某种 JVM 的名称, 比如说是 Zing JVM, 则表示上面第二种含义.

### 1.2	JDK, JRE, JVM 之间的关系

![[JVM, JDK, JRE 之间的关系.png]]

我们利用 JDK (调用 Java API)开发 Java 程序, 编译成字节码或者打包程序. 然后可以用 JRE 则启动一个 JVM 实例, 加载, 验证, 执行 Java 字节码以及依赖库, 运行 Java 程序. 而 JVM 将程序和依赖库的 Java 字节码解析并变成本地代码执行, 产生结果.

## 2	JDK 发展历史时间线

![[JDK 发展历史时间线.png]]





---

# 📚 参考内容

