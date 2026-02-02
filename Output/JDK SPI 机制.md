---
type: permanent
banner: Assets/Banner/pexels-parakit-keng-eos-2435181-4063092.jpg
aliases:
  - JDK SPI
  - Java SPI
---
# 🌐 关键词

JDK, SPI

---

# 🔖 详细解释

## 1	使用流程

**定义接口**: 在核心库中定义接口;

```java
public interface ApplicationStartedListener {  
	void onCompleted();  
}
```

**实现接口**: 提供实现类;

```java
public class DemoListener implements ApplicationStartedListener {  
    @Override  
    public void onCompleted() {  
        System.out.println("DemoListener");  
    }  
}
```

**注册配置**: 在 `META-INF/services/` 目录下, 创建一个以接口全限定名命名的文件, 文件内容为实现类的全限定名;

```
com.ryan.spi.jdk.DemoListener
```

**获取拓展并使用**: 调用 `ServiceLoader.load()` 方法, 获取接口全部的拓展类.

```java
for (int i = 0; i < 3; i++) {  
    ServiceLoader<ApplicationStartedListener> load = ServiceLoader.load(ApplicationStartedListener.class);  
    for (ApplicationStartedListener next : load) {  
        next.onCompleted();  
        // 每次打印均不同, 说明是不同的实例  
        System.out.println(next);  
    }  
}
```

## 2	实现原理

Java 原生 SPI 的原理为:

- 将接口传到 `ServiceLoader.load()` 方法后, 得到了一个内部类的迭代器;
	
- 使用迭代器的 `hasNext()` 方法, 读取 `/META-INF/services/xxx` 这个资源的内容, 然后逐行解析出所有实现类的类路径;
	
- 将所有的类通过 `Class.forName()` 方法实例化.

通过 JDK SPI 每次调用都会进行 读取文件 -> 解析文件 -> 反射创建 这三步, 开销较大. 实现类在迭代器中的位置不固定, 每次获取指定的实现类都需要遍历完所有的实现类.

## 3	JDK SPI 的问题

当使用 `ServiceLoader.load()` 每次都会创建新的实例对象, 容易影响 IO 吞吐和内存消耗;

只能通过迭代器遍历获取, 不能根据某个 key 精确获取某个实现.