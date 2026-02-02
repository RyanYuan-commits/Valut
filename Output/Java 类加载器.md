---
type: permanent
ing: "1"
banner: Assets/Banner/pexels-eberhardgross-1624496.jpg
---
---

**关键词**: Java, 类加载器

---

类加载器是 JVM 的一部分, 负责将类的**字节码文件**加载到**内存**中, 并生成对应的 Class 对象. Java 中的 SPI 机制、类的热部署、Tomcat 隔离都需要借助类加载器来实现.  

![[类加载器加载对象图例.png|900]]


## 1	类加载器的分类

类加载器可以被分为两类, 一类是 Java 代码实现的, 另一类是 JVM 底层实现的.虚拟机底层实现的类加载器的源代码位于 Java 虚拟机的源码中, 实现语言与虚拟机底层语言一致, 比如 Hotspot 使用 C++；它的作用是加载程序运行时的基础类, 保证Java程序运行中基础类被正确的加载, 比如 String 类. JDK中默认提供了多种处于不同渠道的类加载器, 程序员也可以根据自己的需求定制; 所有的 Java 类加载器都需要继承 ClassLoader 类. 

JDK8 及之前的版本默认的类加载器有如下几种: 

- **启动类加载器** (Bootstrap): 加载 Java 中最核心的类. 
    
- **拓展类加载器** (Extension): 加载扩展类库 (%JAVA_HOME%/lib/ext 目录下的 JAR 包, 或者通过 java.ext.dirs 系统属性指定的路径). 
    
- **应用程序类加载器** (Application): 加载用户类路径(CLASSPATH 环境变量指定的路径). 

## 2	启动类加载器

启动类加载器 (Bootstrap ClassLoader) 是由 HotSpot 虚拟机提供, 是用 C++ 编写的类加载器, 它默认负责加载 /jre/lib 目录下的类文件, 比如 rt.jar、tools.jar、resource.jar 等. 

```java
public class Main {  
    public static void main(String[] args) {  
        ClassLoader classLoader = HashMap.class.getClassLoader();  
        System.out.println(classLoader); // null  
    }  
}
```

## 3	拓展类加载器

拓展类加载器和应用程序类加载器都是使用 Java 编写的类加载器, 它们是 sun.misc.Lancher 类的静态内部类, 继承自 URLClassLoader, 具备将字节码文件加载到内存中的功能. 

![[拓展类加载器和应用程序类加载器的继承关系.png|1200]]

拓展类加载器是 JDK 中提供的、使用 Java 编写的类加载器, 它默认加载目录 /jre/lib/ext 下的类文件: 

![[拓展类加载器加载的文件.png]]

它加载的是一些通用但是不那么重要的文件, 比如 Script 运行环境: 

```java
public class Main {  
    public static void main(String[] args) {  
        ClassLoader classLoader = ScriptEnvironment.class.getClassLoader();  
        System.out.println(classLoader); // sun.misc.Launcher$ExtClassLoader@4554617c  
    }  
}
```

## 4	应用程序类加载器

负责加载 ClassPath 下的 jar 包: 

```java
public class Main {  
    public static void main(String[] args) {  
        ClassLoader classLoader = Main.class.getClassLoader();  
        System.out.println(classLoader); // sun.misc.Launcher$AppClassLoader@18b4aac2  
    }  
}
```


---