---
source: https://o.alldu.cn/docs/jvm-%E6%A0%B8%E5%BF%83%E6%8A%80%E6%9C%AF-32-%E8%AE%B2%E5%AE%8C/05-java-%E5%AD%97%E8%8A%82%E7%A0%81%E6%8A%80%E6%9C%AF%E4%B8%8D%E7%A7%AF%E7%BB%86%E6%B5%81%E6%97%A0%E4%BB%A5%E6%88%90%E6%B1%9F%E6%B2%B3/#45-%e6%9f%a5%e7%9c%8b%e6%96%b9%e6%b3%95%e4%bf%a1%e6%81%af
type: input
banner: Assets/Banner/pexels-samkolder-2387877.jpg
---
# 📰 阅读笔记

## 1	字节码简述

Java ByteCode 由单字节的指令构成, 理论最多支持 256 个操作码, Java 使用了 200 个左右.

指令由 **类型前缀 + 操作名称** 构成, 比如 iadd, 代表 integer_add.

根据指令的性质, 可以将指令分为: 

- **栈操作**指令, 包括与局部变量交互的指令
	
- **程序流程控制**指令
	
- **对象操作**指令, 包括方法调用指令
	
- **算术运算**以及**类型转换**指令

## 2	字节码清单解读

### 2.1	获取字节码清单

```java
public class JavaByteCodeDemo {  
    public static void main(String[] args) {  
        JavaByteCodeDemo javaByteCodeDemo = new JavaByteCodeDemo();  
    }  
}

// ====== 编译后 ======

Compiled from "JavaByteCodeDemo.java"
public class com.ryan.demo.JavaByteCodeDemo {
  public com.ryan.demo.JavaByteCodeDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #7                  // class com/ryan/demo/JavaByteCodeDemo
       3: dup
       4: invokespecial #9                  // Method "<init>":()V
       7: astore_1
       8: return
}
```

### 2.2	解读字节码清单

```java
  public com.ryan.demo.JavaByteCodeDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return
```

自动生成的构造方法, 内部的三行代码表示对 super() 的调用.

```java
  public static void main(java.lang.String[]);
    Code:
       0: new           #7                  // class com/ryan/demo/JavaByteCodeDemo
       3: dup
       4: invokespecial #9                  // Method "<init>":()V
       7: astore_1
       8: return
```

创建一个类实例然后返回.

### 2.3	查看文件的常量池信息

通过 javap 的 -v 选项来输出详细信息.

```java
Classfile /D:/code/personal/demo-projects/src/main/java/com/ryan/demo/JavaByteCodeDemo.class
  Last modified 2025年12月17日; size 465 bytes
  SHA-256 checksum be2ba80ca51be1a64fc9cf0b5a3e51ff27ed966ffb9625901e315b582461a990
  Compiled from "JavaByteCodeDemo.java"
public class com.ryan.demo.JavaByteCodeDemo
  minor version: 0
  major version: 61
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #7                          // com/ryan/demo/JavaByteCodeDemo
  super_class: #2                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #2.#3          // java/lang/Object."<init>":()V
   #2 = Class              #4             // java/lang/Object
   #3 = NameAndType        #5:#6          // "<init>":()V
   #4 = Utf8               java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Class              #8             // com/ryan/demo/JavaByteCodeDemo
   #8 = Utf8               com/ryan/demo/JavaByteCodeDemo
   #9 = Methodref          #7.#3          // com/ryan/demo/JavaByteCodeDemo."<init>":()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               LocalVariableTable
  #13 = Utf8               this
  #14 = Utf8               Lcom/ryan/demo/JavaByteCodeDemo;
  #15 = Utf8               main
  #16 = Utf8               ([Ljava/lang/String;)V
  #17 = Utf8               args
  #18 = Utf8               [Ljava/lang/String;
  #19 = Utf8               javaByteCodeDemo
  #20 = Utf8               SourceFile
  #21 = Utf8               JavaByteCodeDemo.java
{
  public com.ryan.demo.JavaByteCodeDemo();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 9: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/ryan/demo/JavaByteCodeDemo;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
         0: new           #7                  // class com/ryan/demo/JavaByteCodeDemo
         3: dup
         4: invokespecial #9                  // Method "<init>":()V
         7: astore_1
         8: return
      LineNumberTable:
        line 12: 0
        line 13: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  args   [Ljava/lang/String;
            8       1     1 javaByteCodeDemo   Lcom/ryan/demo/JavaByteCodeDemo;
}
SourceFile: "JavaByteCodeDemo.java"
```

其中显示了很多关于 class 文件信息: 编译时间, MD5 校验和, 从哪个 . java 源文件编译得来, 符合哪个版本的 Java 语言规范等等. 还可以看到 ACC_PUBLIC 和 ACC_SUPER 访问标志符. ACC_PUBLIC 标志很容易理解:这个类是 public 类, 因此用这个标志来表示.

其中 # 是对常量池的引用.

### 2.4	查看方法信息

```java
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=2, args_size=1
```

- descriptor 部分的小括号内是入参信息/形参信息, 左方括号表述数组; L 表示对象; 后面的 java/lang/String 就是类名称, 小括号后面的 V 则表示这个方法的返回值是 void;
	
- ACC_PUBLIC, ACC_STATIC, 表示 public 和 static.
	
- 还可以看到执行该方法时需要的 stack 深度是 2, 也就是同时在栈上最多会有两个值
	
- 需要在局部变量表中保留 2 个槽位, 还有方法的参数个数为 1:

### 2.5	方法体中的字节码解读

```java
 0: new           #7                  // class com/ryan/demo/JavaByteCodeDemo
 3: dup
 4: invokespecial #9                  // Method "<init>":()V
 7: astore_1
 8: return
```

序号表示这个命令起点位于哪个字节码槽位, 例如, new 占用三个操作, 因为除了操作码, 还存有操作数.

![[Java 字节码槽位示意图.png|800]]





---

# 💭 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?