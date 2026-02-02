---
source: https://time.geekbang.org/column/article/620900?screen=full
created: 2025-11-25
type: input
banner: Assets/Banner/pexels-eliannedipp-4666748.jpg
---
# 📰 阅读笔记

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