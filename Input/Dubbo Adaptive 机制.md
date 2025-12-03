---
source:
created:
type: input
---
# 📰 阅读笔记

```java
Cluster cluster = ExtensionLoader
    // 获取 Cluster 接口对应扩展点加载器
    .getExtensionLoader(Cluster.class)
    // 从 Cluster 扩展点加载器中获取自适应的扩展点
    .getAdaptiveExtension();
```

```java
// org.apache.dubbo.common.extension.ExtensionLoader#getAdaptiveExtension
public T getAdaptiveExtension() {
	checkDestroyed();
	Object instance = cachedAdaptiveInstance.get();
	if (instance == null) {
		if (createAdaptiveInstanceError != null) {
			throw new IllegalStateException(
					"Failed to create adaptive instance: " + createAdaptiveInstanceError.toString(),
					createAdaptiveInstanceError);
		}

		synchronized (cachedAdaptiveInstance) {
			instance = cachedAdaptiveInstance.get();
			if (instance == null) {
				try {
					// 关键代码, 无缓存时构建
					instance = createAdaptiveExtension();
					cachedAdaptiveInstance.set(instance);
				} catch (Throwable t) {
					createAdaptiveInstanceError = t;
					throw new IllegalStateException("Failed to create adaptive instance: " + t.toString(), t);
				}
			}
		}
	}

	return (T) instance;
}
```

```java
// org.apache.dubbo.common.extension.ExtensionLoader#getAdaptiveExtensionClass
// 创建自适应扩展点方法
private T createAdaptiveExtension() {
    try {
        // 这一行从 newInstance 这个关键字便知道这行代码就是创建扩展点的核心代码
        T instance = (T) getAdaptiveExtensionClass().newInstance();
        
        // 这里针对创建出来的实例对象做的一些类似 Spring 的前置后置的方式处理
        instance = postProcessBeforeInitialization(instance, null);
        instance = injectExtension(instance);
        instance = postProcessAfterInitialization(instance, null);
        initExtension(instance);
        return instance;
    } catch (Exception e) {
        throw new IllegalStateException("Can't create adaptive extension " + type + ", cause: " + e.getMessage(), e);
    }
}
                  ↓
// 获取自适应扩展点的类对象
private Class<?> getAdaptiveExtensionClass() {
    // 获取当前扩展点（Cluster）的加载器（ExtensionLoader）中的所有扩展点
    getExtensionClasses();
    // 如果缓存的自适应扩展点不为空的话，就提前返回
    // 这里也间接的说明了一点，每个扩展点（Cluster）只有一个自适应扩展点对象
    if (cachedAdaptiveClass != null) {
        return cachedAdaptiveClass;
    }
    // 这里便是创建自适应扩展点类对象的逻辑，我们需要直接进入没有缓存时的创建逻辑
    return cachedAdaptiveClass = createAdaptiveExtensionClass();
}
                  ↓
// 创建自适应扩展点类对象                  
private Class<?> createAdaptiveExtensionClass() {
    // Adaptive Classes' ClassLoader should be the same with Real SPI interface classes' ClassLoader
    ClassLoader classLoader = type.getClassLoader();
    try {
        if (NativeUtils.isNative()) {
            return classLoader.loadClass(type.getName() + "$Adaptive");
        }
    } catch (Throwable ignore) {
    }
    // 看见这行关键代码，发现使用了一个叫做扩展点源码生成器的类
    // 看意思，就是调用 generate 方法生成一段 Java 编写的源代码
    String code = new AdaptiveClassCodeGenerator(type, cachedDefaultName).generate();
    // 紧接着把源代码传入了 Compiler 接口的扩展点
    // 这个 Compiler 接口不就是我们上一讲思考题刚学过的知识点么
    org.apache.dubbo.common.compiler.Compiler compiler = extensionDirector.getExtensionLoader(
        org.apache.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();
    // 通过调用 compile 方法，也就大致明白了，就是通过源代码生成一个类对象而已
    return compiler.compile(type, code, classLoader);
}
```

在 Dubbo 框架中, 自适应拓展点通过 DCL 以线程安全的方式创建.

每个接口有且仅有一个自适应拓展点;

创建过程是生成了一段 Java 源代码, 然后使用 Compiler 接口编译生成了一个类对象, 是动态生成的.

```java
package org.apache.dubbo.rpc.cluster;
import org.apache.dubbo.rpc.model.ScopeModel;
import org.apache.dubbo.rpc.model.ScopeModelUtil;

// 类名比较特别，是【接口的简单名称】+【$Adaptive】构成的
// 这就是自适应动态扩展点对象的类名

public class Cluster$Adaptive implements org.apache.dubbo.rpc.cluster.Cluster {

    public org.apache.dubbo.rpc.Invoker join(org.apache.dubbo.rpc.cluster.Directory arg0, boolean arg1) throws org.apache.dubbo.rpc.RpcException {
	
        // 如果 Directory 对象为空的话，则抛出异常
        // 一般正常的逻辑是不会走到为空的逻辑里面的，这是一种健壮性代码考虑
        if (arg0 == null) throw new IllegalArgumentException("org.apache.dubbo.rpc.cluster.Directory argument == null");
        // 若 Directory  对象中的 URL 对象为空抛异常，同样是健壮性代码考虑
        if (arg0.getUrl() == null)
            throw new IllegalArgumentException("org.apache.dubbo.rpc.cluster.Directory argument getUrl() == null");
        org.apache.dubbo.common.URL url = arg0.getUrl();
		
        // 这里关键点来了，如果从 url 中取出 cluster 为空的话
        // 则使用默认的 failover 属性，这不恰好就证实了若不配置的走默认逻辑，就在这里体现了
        String extName = url.getParameter("cluster", "failover");
        if (extName == null)
            throw new IllegalStateException("Failed to get extension (org.apache.dubbo.rpc.cluster.Cluster) name from" +
                    " url (" + url.toString() + ") use keys([cluster])");
        ScopeModel scopeModel = ScopeModelUtil.getOrDefault(url.getScopeModel(),
                org.apache.dubbo.rpc.cluster.Cluster.class);
        // 反正得到了一个 extName 扩展点名称，则继续获取指定的扩展点
        org.apache.dubbo.rpc.cluster.Cluster extension =
                (org.apache.dubbo.rpc.cluster.Cluster) scopeModel.getExtensionLoader(org.apache.dubbo.rpc.cluster.Cluster.class)
                .getExtension(extName);
        // 拿着指定的扩展点继续调用其对应的方法
        return extension.join(arg0, arg1);
    }
    // 这里默认抛异常，说明不是自适应扩展点需要处理的业务逻辑
    public org.apache.dubbo.rpc.cluster.Cluster getCluster(org.apache.dubbo.rpc.model.ScopeModel arg0,
                                                           java.lang.String arg1) {
        throw new UnsupportedOperationException("The method public static org.apache.dubbo.rpc.cluster.Cluster org" +
                ".apache.dubbo.rpc.cluster.Cluster.getCluster(org.apache.dubbo.rpc.model.ScopeModel,java.lang.String)" +
                " of interface org.apache.dubbo.rpc.cluster.Cluster is not adaptive method!");
    }
    // 这里默认也抛异常，说明也不是自适应扩展点需要处理的业务逻辑
    public org.apache.dubbo.rpc.cluster.Cluster getCluster(org.apache.dubbo.rpc.model.ScopeModel arg0,
                                                           java.lang.String arg1, boolean arg2) {
        throw new UnsupportedOperationException("The method public static org.apache.dubbo.rpc.cluster.Cluster org" +
                ".apache.dubbo.rpc.cluster.Cluster.getCluster(org.apache.dubbo.rpc.model.ScopeModel,java.lang.String," +
                "boolean) of interface org.apache.dubbo.rpc.cluster.Cluster is not adaptive method!");
    }
    // !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    // 重点推导
    // !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    // 然后继续看看 Cluster，得搞清楚为什么两个 getCluster 方法会抛异常，而 join 方法不抛异常
    // 结果发现接口中的 join 方法被 @Adaptive 注解标识了，但是另外 2 个 getCluster 方法没有被 @Adaptive 标识
    // 由此可以说明一点，含有被 @Adaptive 注解标识的 SPI 接口，是会生成自适应代理对象的
}
```




---

# 💭 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?