---
created: 2025-11-26
type: input
banner: Assets/Banner/pexels-faikackmerd-1025469.jpg
---
# 📰 阅读笔记

Dubbo 有很多像 Protocol, Cluster, LoadBalance 这样的拓展点, 在框架启动时, 我们不知道具体使用哪个实例, 而是要等到实际调用时根据方法的参数, URL 的配置等信息才能做出.

Adaptive 机制的核心思想: 动态生成一个适配器类, 在**运行时**更加 URL 或参数中的 "拓展点" 名来动态选择和使用正确的拓展点实现.

核心是 @Adaptive 注解, 当在方法上使用时, Dubbo 的 ExtensionLoader 会在运行时动态的生成这个接口的适配器实现类.

例如 Protocol 接口:

```java
@SPI("dubbo")
public interface Protocol {
    @Adaptive // 关键注解！
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;
    
    @Adaptive // 关键注解！
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;
    // ...
}
```

Dubbo 自动生成一个名为 Protocol$Adaptive 的类, 大致逻辑为:

```java
public class Protocol$Adaptive implements Protocol {
    
    public <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException {
        // 1. 从URL中获取决定协议类型的参数，默认是 @SPI 注解的值 "dubbo"
        //    参数名通常从 @Adaptive 注解的 value 属性获取，如果没写，则根据类名推导（如Protocol取`protocol`）。
        String extName = (url.getProtocol() == null ? "dubbo" : url.getProtocol());
        
        // 2. 使用 ExtensionLoader 根据 extName 获取真正的 Protocol 实现
        Protocol extension = ExtensionLoader.getExtensionLoader(Protocol.class).getExtension(extName);
        
        // 3. 调用具体实现的 refer 方法
        return extension.refer(type, url);
    }
    
    // ... export 方法类似
}
```

当开发者调用有 @Adaptive 方法的注解时, 适配器从传入的 URL 中提取出 Protocol 的值, 如果是 dubbo 会去找 DubboProtocol, 如果是 zookeeper, 找 RegistryProtocol, 然后调用最终实现的 refer 方法.

- **SPI 机制** 提供了扩展点的**静态发现和管理能力**. 它定义了有哪些实现以及如何通过名称找到它们; 它是整个扩展体系的仓库.
- **Adaptive 机制** 提供了扩展点的**动态选择能力**. 它解决了在运行时应该使用哪个实现的问题. 它是整个扩展体系的调度中心.

如果我获取的是一个 Adaptive 的类, 那它没有被注解标注的方法会调用 SPI 指定的实现类, 而如果有的话, 会根据参数来调用.


---

# 💭 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?