---
type: permanent
banner: Assets/Banner/pexels-maxravier-3331094.jpg
---
---

**关键词**: Java, 类加载器, 双亲委派机制 

---

## 1	应用场景

**隔离加载类**: 在某些情况下需要隔离加载的类, 例如在服务器中多个应用的类需要相互隔离, 避免类名冲突. 这时就需要打破双亲委派机制, 使得每个应用都有自己的类加载器. 

**修改类加载方式**: 在某些情况下需要修改类的加载方式, 如热部署. 当我们的代码发生改变时, 我们希望能够重新加载类, 而不需要重启 JVM. 这时就需要打破双亲委派机制, 使得我们能够控制类的加载. 

**加载不同路径下的类**: 在某些情况下希望能够加载不同路径下的同名类, 这时就需要打破双亲委派机制, 使得我们能够加载不同路径下的类. 

## 2	实现原理

Java 中, 只有一个类的全限定名 + 类加载器相同时候, 才会被认为是同一个类, 所以可以通过自定义类加载器或 Osgi 框架的方式实现隔离.

## 3	使用自定义类加载器

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

一个 Tomcat 实例中可以运行多个 Web 应用, 多个 Web 应用之间使用的类应该是相互隔离的, 为了实现这种效果, Tomcat 为每一个应用都分配一个独立的自定义类加载器.

---