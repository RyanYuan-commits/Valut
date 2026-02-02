---
type: permanent
banner: Assets/Banner/pexels-yuyang-zhang-1354920916-25785552.jpg
---
---

**关键词**: Dubbo, SPI

---

[[JDK SPI 机制|JDK SPI]] 没有缓存能力且无法通过名称精准匹配实例, 为了弥补它的不足, Dubbo 也定义出了一套自己的 SPI 机制, 能实现 O(1) 时间复杂度获取实例对象, 并且缓存实例, 做到按需加载.

## 1	使用方式

定义一个 `IDemoSpi` 接口, 在该接口上添加 `@SPI` 注解:

```java
@SPI
public interface IDemoSpi {  
    int getDefaultPort();  
}
```

定义 `CustomSpi` 实现 `IDemoSpi` 接口

```java
public class CustomSpi implements IDemoSpi {  
    @Override  
    public int getDefaultPort() {  
        return 8080;  
    }  
}
```

在 `/META-INF/services/xxx.xxx.IDemoSpi` 中添加别名 + 类路径

```
customSpi=com.ryan.spi.dubbo.CustomSpi
```

通过 `ExtensionLoader.getExtension()` 方法传入别名来获取实现类.

```java
ApplicationModel applicationModel = ApplicationModel.defaultModel();  
ExtensionLoader<IDemoSpi> extensionLoader = applicationModel.getExtensionLoader(IDemoSpi.class);  
IDemoSpi customSpi = extensionLoader.getExtension("customSpi");  
System.out.println(customSpi.getDefaultPort()); // Output: 8080  
  
IDemoSpi customSpi1 = extensionLoader.getExtension("customSpi");  
System.out.println(customSpi == customSpi1); // Output: true
```