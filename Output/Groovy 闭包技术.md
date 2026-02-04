---
type: permanent
banner: Assets/Banner/pexels-ken-cheung-3355734-5574638.jpg
---
---

**关键词**: Groovy, Closure 

---

## 1	什么是闭包?

闭包是一个匿名的代码块, 在 Groovy 中, 闭包是 Closure 类的实例. 其包含参数, 箭头和可执行的代码, 参数是可选的, 如果有多个, 使用 `,` 分割. 可以像执行一个函数一样去执行闭包:

```groovy
// 这是一个闭包，它接受一个参数并打印它
def myClosure = { name ->
    println("Hello, ${name}")
}

// 像调用函数一样调用它
myClosure("Groovy")
myClosure.call("Groovy")
```


## 2	Groovy 针对闭包的语法简化

### 2.1	作为参数的语法简化

在 Groovy 中, 当函数的**最后一个参数**是闭包, 可以将闭包写在括号外面;

```groovy
// 传统写法
someMethod(arg1, arg2, { closureBody })

// Groovy 语法糖写法
someMethod(arg1, arg2) { closureBody }
```

所以在 Gradle 构建脚本中类似 `dependencies { ... }` 和 `tasks { ... }` 的形式, 其实是传递了闭包的函数调用.

### 2.2	隐式参数

如果闭包只有一个参数时, 可以省略参数和 `->` 符号, 这个唯一的参数会被隐式的命名为 `it`;

```groovy
// 传统写法，显式参数
def printNumber = { number -> println("Number: ${number}") }

// 简写，使用隐式参数 'it'
def printNumberIt = { println("Number: ${it}") }

printNumberIt(42) // 输出: Number: 42
```

比如, 在 `subprojects` 中, 可以通过 `it.name` 来获取当前配置项目的名称.

## 3	委托对象

在闭包内部, 每当调用一个方法或者访问一个属性, Groovy 会首先尝试在闭包的**委托对象**上查找; 可以通过 `c.delegate = xxx` 来指定委托对象; 

当在 Gradle 构建脚本中写 `dependencies { ... }` 时, 闭包委托对象的类型是 `DependencyHandler`, 所以你在 `{ ... }` 可以直接的调用 `implemention`, `testImplementions` 等方法.

```groovy
class Printer {
    void print(Closure c) {
	    // 指定闭包的委托对象是 this
        c.delegate = this
        c.resolveStrategy = Closure.DELEGATE_FIRST
        c.call()
    }
    
    void printLine(String s) {
        println("--- ${s} ---")
    }
}

def printer = new Printer()

printer.print {
    printLine("User Info:")
    printLine("Name: ${name}")
    printLine("Age: ${age}")
}
```

---