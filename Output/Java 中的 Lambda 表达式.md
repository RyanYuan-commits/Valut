---
type: permanent
---
# 🌐 核心观点

Lambda 表达式是 JDK8 引入的新特性, 提供了一种简洁的方式来表示匿名函数, 用于实现 **函数式接口**;

---

# 🔖 详细解释

## 1	核心特点

Lambda 表达式没有名称, 可以直接作为参数传递; 显著减少了编写匿名内部类需要的样板代码;

它们只能用于实现函数式接口, 使代码能够接受不同的函数作为参数, 支持函数式编程;

是 Java Stream API 中进行数据处理的核心机制.

## 2	编码案例

在 JDK8 之前, 使用匿名内部类来实现排序接口

```java
Comparator<String> legacyComparator = new Comparator<String>() {
	@Override
	public int compare(String s1, String s2) {
		// 按字符串长度降序排序
		return s2.length() - s1.length();
	}
};
```

而使用 Lambda 表达式实现:

```java
Collections.sort(frameworks, (s1, s2) -> s2.length() - s1.length());
```

使用匿名内部类时, 编译器会生成一个单独的 .class 文件, 相对较重. 而 Lambda 表达式的出现不仅解决了语法冗长的问题, 也底层实现上进行了彻底的优化.

### 2.1	核心机制: invokedynamic 指令

invokedynamic 是 JDK7 引入的一条新的 JVM 字节码指令, 它最初是为了支持运行在 JVM 上的动态语言 (如 JRuby, Groovy) 而设计的; 与其他的调用指令, 如 invokestatic, invokevirtual 不同, invokedynamic 的方法调用的链接过程推迟到运行时.

当编译器遇到 Lamda 表达式时, 它并不会像匿名内部类那样立即生成一个新的类, 而是会执行如下操作:

- **生成一个 "脱糖" 方法 (Desugaring)**: 编译器会将 Lambda 表达式的主体代码提取出来, 生成一个静态私有方法(如果 Lambda 捕获了实例变量, 则会生成一个实例私有方法), 这个方法包含了 Lambda 的具体逻辑, 方法名通常是 `lambda$..` 的形式.
	
- **生成 invokedynamic 指令**: 在原来使用 Lambda 表达式的地方, 编译器会插入一条 invokedynamic 指令. 这条指令告诉 JVM: "这里需要一个实现了某个函数式接口的对象, 但具体如何创建这个对象, 请在运行时决定."

这个 "在运行时决定" 的过程, 由一个特殊的 Bootstrap Method 来完成对于 Lambda 表达式, 这个引导方法通常是 LambdaMetafactory.metafactory().

#### 2.1.1	Execution Process

**首次执行**: 当 `invokedynamic` 指令第一次被执行时, JVM 会调用其指定的引导方法 `LambdaMetafactory.metafactory`;

**引导方法工作**: `LambdaMetafactory` 会接收到 JVM 传来的所有必要信息, 包括: 

- Lambda 表达式要实现的函数式接口是什么, 如 `Runnable`.
- 接口中需要实现的方法签名是什么, 如 `run()`.
- 编译器生成的那个私有静态/实例方法(即 Lambda 的主体代码)的方法句柄。

**动态生成类并创建实例**: `LambdaMetafactory` 会在内存中**动态地生成一个类**. 这个类实现了目标函数式接口, 并且其接口方法的实现就是去调用编译器之前生成的那个私有方法. 然后, 它会创建一个该类的实例.

**链接调用点**: 引导方法返回一个 `CallSite` 对象, 这个对象会和 `invokedynamic` 所在的位置进行永久链接. `CallSite` 内部包含了创建 Lambda 实例的逻辑.

**后续执行**: 之后再次执行到这条 `invokedynamic` 指令时, JVM 不会再调用引导方法, 而是直接使用已经链接好的 `CallSite` 来快速创建或返回 Lambda 实例. 对于没有捕获任何外部变量的 Lambda, `LambdaMetafactory` 足够智能, 可以返回一个**单例实例**, 从而达到极高的性能.

## 3	Effective Final

当 Lambda 表达式被创建时, 它并不是获取了外部局部变量的内存地址, 而是像拍快照一样, **复制并持有了那个变量当时的值**. 

定义 Lambda 的方法执行完后, 其局部变量就会被销毁. 但 Lambda 对象本身可能还存活(比如在另一个线程里). 如果 Lambda 持有的是变量的引用, 那么它将指向一块被回收的内存, 导致程序崩溃. 因此, 只能复制值.

既然 Lambda 内部持有的是一个值的 "快照", 如果外部的原始变量还能被修改, 那么 "快照" 里的值就和外面的新值不一致了. 这会导致代码的行为和程序员的直觉严重冲突, 产生难以调试的 Bug. Lambda 经常被用于多线程, 如果一个变量可以被主线程修改, 同时又被 Lambda 所在的子线程读取, 就会产生数据竞争, 导致结果完全不可预测.

---

# 📚 参考内容

