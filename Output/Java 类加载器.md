---
type: permanent
ing: "1"
---
# 🌐 核心观点

用一两句话概括这个原子化的想法.

---

# 🔖 详细解释

类加载器(Class Loader)是 JVM 的一部分, 负责将类的**字节码文件**加载到**内存**中, 并生成对应的 Class 文件. 

![[类加载器加载对象图例.png|900]]

Java 中的 SPI 机制、类的热部署、Tomcat 隔离都需要借助类加载器来实现. 

## 1	类加载器的分类

类加载器可以被分为两类, 一类是 Java 代码实现的, 另一类是 JVM 底层实现的.

- 虚拟机底层实现: 其源代码位于 Java 虚拟机的源码中, 实现语言与虚拟机底层语言一致, 比如 Hotspot 使用 C++；它的作用是加载程序运行时的基础类, 保证Java程序运行中基础类被正确的加载, 比如 String 类. 
    
- JDK 中默认提供或者自定义的类加载器: JDK中默认提供了多种处于不同渠道的类加载器, 程序员也可以根据自己的需求定制；所有的 Java 类加载器都需要继承 ClassLoader 类. 

JDK8 及之前的版本默认的类加载器有如下几种: 

- 启动类加载器 (Bootstrap): 加载 Java 中最核心的类. 
    
- 拓展类加载器 (Extension): 加载扩展类库 (%JAVA_HOME%/lib/ext 目录下的 JAR 包, 或者通过 java.ext.dirs 系统属性指定的路径). 
    
- 应用程序类加载器 (Application): 加载用户类路径(CLASSPATH 环境变量指定的路径). 

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

## 5	双亲委派机制

### 5.1	什么是双亲委派机制？

![[双亲委派机制示意图.png|700]]

在 Java 中, 除 Bootstrap 启动类加载器以外, 每个类加载器都有一个父类加载器, 当一个类加载器收到加载请求后: 

- 类加载请求先委派给父加载器处理;
	
- 父加载器无法完成时(在自己的路径下找不到类), 才由子加载器尝试加载.

双亲委派机制避免了恶意代码替换 JDK 中的核心类库, **确保核心类库的完整性和安全性**, **同时也避免了类被重复加载**. 


>[!question] 如何通过编码的方式使用类加载器去加载一个类呢？
>使用 Class.forName() 方法, 通过类的全限定名的方式去加载类, 这种方式会使用类加载器去加载指定的类. 
>获取到类加载器之后, 可以通过这个类加载器实例的 loadClass 方法来加载类. 

### 5.2	ClassLoader.loadClass 方法

```java
// java.lang.ClassLoader#loadClass(java.lang.String, boolean)
protected Class<?>  loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // 检查类是否已经被当前加载器加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        // 检查父类加载器是否加载了这个类
                        c = parent.loadClass(name, false);
                    } else {
                        // 如果没有父类加载器, 检查 BootStrap 加载器是否加载了这个类
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
					// ......
                }
				
                if (c == null) {
					// 当前类加载器的加载逻辑
                }
            }
            // 检查是否需要触发解析阶段
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

上面的是类加载器的抽象父类 ClassLoader 实现的类加载方法, 当发现自己没有加载这个类之后, 会调用父类加载器的 loadClass 方法; 相当于一个向上递归调用的过程, 递归的终点为: 

- 直到执行完 findBootstrapClassOrNull 还是发现类没有被加载过, 则从递归的终点开始尝试加载类;
	
- 某一个类加载器执行 findLoadedClass 发现自己加载过这个类, 直接返回 Class 实例.

### 5.3	打破双亲委派机制

虽然双亲委派机制有着确保核心类库的完整性和安全性和同时也避免了类被重复加载的作用, 但是在以下几种情况下, 仍然需要去打破双亲委派机制. 

- **隔离加载类**: 在某些情况下需要隔离加载的类, 例如在服务器中多个应用的类需要相互隔离, 避免类名冲突. 这时就需要打破双亲委派机制, 使得每个应用都有自己的类加载器. 
	
- **修改类加载方式**: 在某些情况下需要修改类的加载方式, 例如热部署. 当我们的代码发生改变时, 我们希望能够重新加载类, 而不需要重启 JVM. 这时就需要打破双亲委派机制, 使得我们能够控制类的加载. 
	
- **加载不同路径下的类**: 在某些情况下希望能够加载不同路径下的同名类, 这时就需要打破双亲委派机制, 使得我们能够加载不同路径下的类. 

可以通过自定义类加载器继承 ClassLoader 并重写 loadClass 方法 或者 通过线程上下文类加载器和 Osgi 框架等方式实现. 

#### 5.3.1	自定义类加载器

一个 Tomcat 中是可以运行多个 Web 应用的, 如果这两个应用之间出现了相同限定名的类, 比如 Servlet 类, Tomcat 要保证这两个类都能被加载, 而且使用的时候应该是不同的类, 如果不打破双亲委派机制后, 两个应用中相同限定名的类就无法同时被加载了, Tomcat 中为每个应用生成了一个独立的类加载器去加载对应的类

```java
// 通过继承 ClassLoader 实现的自定义类加载器: 
public class MyClassLoader extends ClassLoader {  
  
    final String basePath;  
  
    public MyClassLoader(String basePath) {  
        this.basePath = basePath;  
    }  
  
    private byte[] loadSource(String path) {  
        String realPath = basePath + File.separator + path;  
        try (FileInputStream inputStream = new FileInputStream(realPath)) {  
            int available = inputStream.available();  
            ByteBuffer allocate = ByteBuffer.allocate(available);  
            FileChannel channel = inputStream.getChannel();  
            channel.read(allocate);  
            return allocate.array();  
        } catch (IOException exception) {  
            System.out.println("load class failed");  
        }  
        return null;  
    }  
  
    @Override  
    public Class<?> loadClass(String name) throws ClassNotFoundException {  
        if (name.startsWith("java.")) {  
            return super.loadClass(name);  
        }  
        synchronized (getClassLoadingLock(name)) {  
            byte[] bytes = loadSource(name);  
            if (bytes == null) {  
                return super.loadClass(name);  
            }  
            return defineClass(null, bytes, 0, bytes.length);  
        }  
    }  
}

public class Main {  
    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {  
        MyClassLoader myClassLoader = new MyClassLoader("/Users/ryan/Documents/code/stu-demo/my-platform/target/classes/com/ryan");  
        Class<?> clazz = myClassLoader.loadClass("A.class");  
        System.out.println(clazz.getClassLoader());  
        Class<?> aClass = Class.forName("com.ryan.A");  
        System.out.println(aClass.getClassLoader());  
    }  
}  
  
class A {  
    static {  
        System.out.println("A 类被加载了");  
    }  
}
```

输出结果: 

```
com.ryan.classloader.MyClassLoader@28d93b30
A 类被加载了
sun.misc.Launcher$AppClassLoader@18b4aac2
```

上面可以看到, A 类被加载了两次, 这是因为在 Java 虚拟机中, 只有相同的类加载器 + 相同的类限定名才会被识别成同一个类, 即使类限定名相同, 如果使用的是不同的类加载器, 也会被识别为不同的类. 

#### 5.3.2	线程上下文类加载器

JDBC 中使用了 DriverManager 的类来管理不同的数据库的驱动, 比如 Mysql 驱动, Oracle 驱动. 

假设我们执行 DriverManager.getConnection() 方法来获取与 MySQL 的连接, 如果我们没有显示的在代码中去注册驱动, 就会去调用 Java 的 SPI 机制来查找 classpath 中的 Mysql 驱动. 

SPI 机制是 JDK 内置的一种服务发现机制, 比如我们要制作一个 MySQL 的驱动程序, 就要去实现 java.sql.Driver 接口之中规定的方法, 这时候, 要在 META-INF/service 这个位置中存放一个名字为 java.sql.Driver 的文件, 文件中存放的是实现了这个接口的类的全限定名, 比如 MySQL 驱动的这个文件中就存放的是 com.mysql.cj.jdbc.Driver 即, 类中实现了那个接口的类的全限定名. 

ServiceLoader 中依靠 SPI 机制, 可以从符合上面规范的包中读取并且 **实例化** 驱动程序类, 并将其存放在 DriverManager 中的 registeredDrivers 属性中；

```java
private static void loadInitialDrivers() {
    AccessController.doPrivileged(new PrivilegedAction<Void>() {
        public Void run() {
            ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
            Iterator<Driver> driversIterator = loadedDrivers.iterator();
            try {
                while (driversIterator.hasNext()) {
                    driversIterator.next();
                }
            } catch (Throwable t) {
                // Ignore exception during driver loading
            }
            return null;
        }
    });
}
```

ServiceLoader 是 Java SPI 机制的核心工具, 负责加载驱动类. 在 JDBC 中, 它通过以下逻辑找到 META-INF/services/java.sql.Driver 文件并加载对应的驱动实现. 

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}
```

getContextClassLoader() 方法会返回当前线程的上下文类加载器, 这个类加载器就是应用程序类加载器. 

```java
public class Main {  
    public static void main(String[] args) {  
        TestThread testThread = new TestThread();  
        testThread.start();  // sun.misc.Launcher$AppClassLoader@18b4aac2
    }  
}  
class TestThread extends Thread {  
    @Override  
    public void run() {  
        System.out.println(this.getContextClassLoader());  
    }  
}
```

当一个线程被创建后它的 getContextClassLoader 方法获取到的就是 AppClassLoader. 

---

# 📚 参考内容

