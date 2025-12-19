---
type: permanent
---
# 🌐 核心观点

用一两句话概括这个原子化的想法.

---

# 🔖 详细解释

类加载器(Class Loader)是 JVM 的一部分, 负责将类的**字节码文件**加载到**内存**中, 并生成对应的 Class 文件. 

![[类加载器加载对象图例.png|900]]

Java 中的 SPI 机制、类的热部署、Tomcat 隔离都需要借助类加载器来实现. 

## 类加载器的分类

类加载器可以被分为两类, 一类是 Java 代码实现的, 另一类是 JVM 底层实现的. 

- 虚拟机底层实现: 其源代码位于 Java 虚拟机的源码中, 实现语言与虚拟机底层语言一致, 比如 Hotspot 使用 C++；它的作用是加载程序运行时的基础类, 保证Java程序运行中基础类被正确的加载, 比如 String 类. 
    
- JDK 中默认提供或者自定义的类加载器: JDK中默认提供了多种处于不同渠道的类加载器, 程序员也可以根据自己的需求定制；所有的 Java 类加载器都需要继承 ClassLoader 类. 

JDK8 及之前的版本默认的类加载器有如下几种: 

- 启动类加载器(Bootstrap): 加载 Java 中最核心的类. 
    
- 拓展类加载器(Extension): 加载扩展类库 (%JAVA_HOME%/lib/ext 目录下的 JAR 包, 或者通过 java.ext.dirs 系统属性指定的路径). 
    
- 应用程序类加载器(Application): 加载用户类路径(CLASSPATH 环境变量指定的路径). 

## 启动类加载器

启动类加载器(Bootstrap ClassLoader)是由 HotSpot 虚拟机提供的, 使用 C++ 编写的类加载器, 它默认负责加载 /jre/lib 目录下的类文件, 比如 rt.jar、tools.jar、resource.jar 等. 

```java
public class Main {  
    public static void main(String[] args) {  
        ClassLoader classLoader = HashMap.class.getClassLoader();  
        System.out.println(classLoader); // null  
    }  
}
```



## 拓展类加载器

拓展类加载器和应用程序类加载器都是 JDK 提供的, 使用 Java 编写的类加载器, 它们是 sun.misc.Lancher 类的静态内部类, 继承自 URLClassLoader, 具备将字节码文件加载到内存中的功能. 

![[拓展类加载器和应用程序类加载器的继承关系.png]]

拓展类加载器(Extension Class Loader)是 JDK 中提供的、使用 Java 编写的类加载器, 它默认加载目录 /jre/lib/ext 下的类文件: 

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

### 3.2 应用程序类加载器

负责加载 ClassPath 下的 jar 包: 

```java
public class Main {  
    public static void main(String[] args) {  
        ClassLoader classLoader = Main.class.getClassLoader();  
        System.out.println(classLoader); // sun.misc.Launcher$AppClassLoader@18b4aac2  
    }  
}
```

## 2 双亲委派机制

### 2.1 什么是双亲委派机制？

双亲委派机制是 Java 中类加载器的一个重要机制. 在 Java 中, 处理 Bootstrap 启动类加载器以外, 每个 ClassLoader 都有一个父 ClassLoader；

当一个 ClassLoader 收到类加载的请求的时候, 会将加载请求委派给父 ClassLoader, 如果父 ClassLoader 能够或者已经完成了这个类的加载, 将 Class 对象直接返回；只有当最顶层的 ClassLoader 都无法完成此类的加载, 才会逐层的尝试由自己来加载. 

双亲委派机制避免了恶意代码替换 JDK 中的核心类库. 比如 String, **确保核心类库的完整性和安全性**; **同时也避免了类被重复加载**. 

用一句话来说, 双亲委派机制就是: 当一个类接收到加载类的任务的时候, 会自底向上查找类是否被加载过, 如果被加载过就直接返回, 如果未被加载过, 就会自顶向下的去进行类的加载. 

![[双亲委派机制示意图.png|700]]

>[!question] 如何通过编码的方式使用类加载器去加载一个类呢？
>使用 Class.forName() 方法, 通过类的全限定名的方式去加载类, 这种方式会使用类加载器去加载指定的类. 
>获取到类加载器之后, 可以通过这个类加载器实例的 loadClass 方法来加载类. 

```java
public class Main {  
    public static void main(String[] args) throws ClassNotFoundException {  
        Main.class.getClassLoader().loadClass("com.ryan.A");  
        Class.forName("com.ryan.A");  
    }  
}  
  
class A {  
    static {  
        System.out.println("A 类被加载了");  // 只会输出一次
    }  
}
```

### ClassLoader.loadClass 方法

```java
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
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
                // 如果类未被父加载器加载, 也未从启动类路径中找到, 则调用当前加载器的 findClass(name) 方法进行加载
                if (c == null) {
                    long t1 = System.nanoTime();
                    // 类需要重写的抽象方法, 用于定义自定义加载逻辑, 比如从网络、文件系统或其他资源中加载类. 
                    c = findClass(name);

                    // 记录加载性能(使用 sun.misc.PerfCounter 收集统计数据)
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            // 检查是否需要出发解析阶段
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

上面的代码就是 ClassLoader 中实现的类加载方法, 方法中实现了双亲委派机制, 具体来说就是当发现自己没有加载这个类之后, 就会去调用 parent 的 loadCLass 方法；

这相当于一个向上递归调用的过程, 也就是前面双亲委派机制提到过的, ==向上查找是否被加载过的过程==, 递归的终点有这两种: 

1. 直到执行完 findBootstrapClassOrNull 还是发现类没有被加载过
	
2. 某一个类加载器执行 findLoadedClass 发现自己加载过这个类
	
第一种情况发生后, 会从递归终点开始尝试调用 findClass 来加载类, 也就是==向下尝试加载的过程==；而如果发现类已经被加载过, 也就是第二种情况, 直接返回加载过的类. 当上面的方法执行完, 其实就完成了类的生命周期中的**类加载阶段** 和 **连接阶段** 或者 **连接阶段的一部分**; 如果 resolve 变量为 false, 就不会触发连接阶段中的 **解析** 阶段, 也就是将符号引用转换为直接引用的阶段, 这个阶段会延后到真正使用类的时候再开始. 

### 打破双亲委派机制

虽然双亲委派机制有着确保核心类库的完整性和安全性和同时也避免了类被重复加载的作用, 但是在以下集中情况下, 我们仍然需要去打破双亲委派机制. 

1. ==隔离加载类==: 在某些情况下, 我们需要隔离加载的类, 例如在服务器中多个应用的类需要相互隔离, 避免类名冲突. 这时就需要打破双亲委派机制, 使得每个应用都有自己的类加载器. 
	
2. ==修改类加载方式==: 在某些情况下, 我们需要修改类的加载方式, 例如热部署. 当我们的代码发生改变时, 我们希望能够重新加载类, 而不需要重启 JVM. 这时就需要打破双亲委派机制, 使得我们能够控制类的加载. 
	
3. ==加载不同路径下的类==: 在某些情况下, 我们希望能够加载不同路径下的同名类, 这时就需要打破双亲委派机制, 使得我们能够加载不同路径下的类. 

那如何打破双亲委派机制呢？既然双亲委派机制是通过 ClassLoader 中的 loadClass 方法实现的, 我们就可以通过自定义类加载器继承 ClassLoader 并重写 loadClass 方法来实现. 除了这种方法之外, 还可以通过线程上下文类加载器和 Osgi 框架等方式实现. 

#### 自定义类加载器

一个 Tomcat 中是可以运行多个 Web 应用的, 如果这两个应用之间出现了相同限定名的类, 比如 Servlet 类, Tomcat 要保证这两个类都能被加载, 而且使用的时候应该是不同的类, 如果不打破双亲委派机制后, 两个应用中相同限定名的类就无法同时被加载了, Tomcat 中为每个应用生成了一个独立的类加载器去加载对应的类

通过继承 ClassLoader 实现的自定义类加载器: 

```java
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
```

Main 方法与 A 类

```java
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

上面可以看到, A 类被加载了两次, 这是因为在 Java 虚拟机中, 只有 ==相同的类加载器 + 相同的类限定名== 才会被识别成同一个类, 即使类限定名相同, 如果使用的是不同的类加载器, 也会被识别为不同的类. 

#### 线程上下文类加载器

JDBC 中使用了 DriverManager 的类来管理不同的数据库的驱动, 比如 Mysql 驱动, Oracle 驱动. 

假设我们执行 DriverManager.getConnection() 方法来获取与 MySQL 的连接, 如果我们没有显示的在代码中去注册驱动, 就会去调用 Java 的 SPI 机制来查找 classpath 中的 Mysql 驱动. 

SPI 机制是 JDK 内置的一种服务发现机制, 比如我们要制作一个 MySQL 的驱动程序, 就要去实现 java.sql.Driver 接口之中规定的方法, 这时候, 要在 META-INF/service 这个位置中存放一个名字为 java.sql.Driver 的文件, 文件中存放的是实现了这个接口的类的全限定名, 比如 MySQL 驱动的这个文件中就存放的是 com.mysql.cj.jdbc.Driver 即, 类中实现了那个接口的类的全限定名. 

ServiceLoader 中依靠 SPI 机制, 可以从符合上面规范的包中读取并且 **实例化** 驱动程序类, 并将其存放在 DriverManager 中的 registeredDrivers 属性中；

**核心代码(DriverManager 部分实现)**
```java
static {
    loadInitialDrivers();
}

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

**核心代码(ServiceLoader 使用类加载器)**
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

