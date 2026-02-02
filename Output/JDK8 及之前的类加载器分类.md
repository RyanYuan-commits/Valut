---
type: permanent
banner: Assets/Banner/pexels-eberhardgross-1624496.jpg
---
---

**关键词**: Java, 类加载器

---

类加载器是 JVM 的一部分, 负责将类的**字节码文件**加载到**内存**中, 并生成对应的 Class 对象. Java 中的 SPI 机制、类的热部署、Tomcat 隔离都需要借助类加载器来实现.  

![[类加载器加载对象图例.png|900]]


## 1	启动类加载器

启动类加载器 (Bootstrap ClassLoader) 是由 HotSpot 虚拟机提供, 使用 C++ 编写的类加载器, 默认负责加载 /jre/lib 目录下的类文件, 比如 rt.jar、tools.jar、resource.jar 等. 

```java
public class Main {  
    public static void main(String[] args) {  
        ClassLoader classLoader = HashMap.class.getClassLoader();  
        System.out.println(classLoader); // null  
    }  
}
```

## 2	拓展类加载器

拓展类加载器和应用程序类加载器都是使用 Java 编写的类加载器, 它们是 `sun.misc.Lancher` 类的静态内部类, 继承自 URLClassLoader, 具备将字节码文件加载到内存中的功能. 

![[拓展类加载器和应用程序类加载器的继承关系.png|900]]

拓展类加载器是 JDK 中提供的、使用 Java 编写的类加载器, 它默认加载目录 /jre/lib/ext 下的类, 是一些通用但是不那么重要的文件, 比如 Script 运行环境:

```java
public class Main {  
    public static void main(String[] args) {  
        ClassLoader classLoader = ScriptEnvironment.class.getClassLoader();  
        System.out.println(classLoader); // sun.misc.Launcher$ExtClassLoader@4554617c  
    }  
}
```

## 3	应用程序类加载器

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