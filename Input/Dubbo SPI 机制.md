---
source: https://time.geekbang.org/column/article/620900?screen=full
created: 2025-11-25
type: input
banner: Assets/Banner/pexels-eliannedipp-4666748.jpg
---
# 📰 阅读笔记

SPI, Service Provider Interface Interface, 服务提供者接口, 常用于拓展点提供场景.

## 1	JDK SPI

Java 原生 SPI 的原理为:

- 将接口传到 ServiceLoader.load 方法后, 得到了一个内部类的迭代器;
	
- 使用迭代器的 hasNext 方法, 读取 "/META-INF/services/xxx" 这个资源的内容, 然后逐行解析出所有实现类的类路径;
	
- 将所有的类路径通过 Class.forName 的方式实例化.

通过 JDK SPI 每次调用都会进行 读取文件 -> 解析文件 -> 反射创建 这三步, 开销较大. 实现类在迭代器中的位置不固定, 每次获取指定的实现类都需要遍历完所有的实现类.

## 2	Dubbo SPI

相比于 JDK SPI, 的两个缺点, Dubbo 使用缓存 + hash 的方式构建 SPI.

```java
// com.ryan.spi.dubbo.DubboSpiDemo#main
public static void main(String[] args) {
	ApplicationModel applicationModel = ApplicationModel.defaultModel();
	ExtensionLoader<IDemoSpi> extensionLoader = applicationModel.getExtensionLoader(IDemoSpi.class);
	IDemoSpi customSpi = extensionLoader.getExtension("customSpi");
	System.out.println(customSpi.getDefaultPort()); // Output: 8080

	IDemoSpi customSpi1 = extensionLoader.getExtension("customSpi");
	System.out.println(customSpi == customSpi1); // Output: true
}

// com.ryan.spi.dubbo.IDemoSpi
@SPI  
public interface IDemoSpi {  
    int getDefaultPort();  
}

// com.ryan.spi.dubbo.CustomSpi
public class CustomSpi implements IDemoSpi {  
    @Override  
    public int getDefaultPort() {  
        return 8080;  
    }  
}

// /META-INF/dubbo/internal/com.ryan.spi.dubbo.IDemoSpi
customSpi=com.ryan.spi.dubbo.CustomSpi
```

定义接口, 添加 SPI 注解, 在资源文件中注册, 获取到实例是单例的, 且可以自由的配置 key.

---

# 💭 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?