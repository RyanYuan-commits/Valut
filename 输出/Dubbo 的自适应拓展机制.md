---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
aliases:
  - Dubbo Adaptive
---
---

**关键词**: Dubbo, 自适应拓展, Adaptive 

---

[[Dubbo SPI 机制]] 提供了对于拓展点的静态发现和管理能力, 而在实际应用场景中, 在**编译时**并不能确定要使用拓展点的哪个实现, 而是要在运行时**动态决定**.

## 1	使用说明

```java
protected ChannelHandler wrapInternal(ChannelHandler handler, URL url) {  
    return new MultiMessageHandler(new HeartbeatHandler(url.getOrDefaultFrameworkModel()  
            .getExtensionLoader(Dispatcher.class)  
            .getAdaptiveExtension()  
            .dispatch(handler, url)));  
}
```

通过 `ExtensionLoader.getAdaptiveExtension()` 方法获取该接口的自适应拓展实现, 后续会通过 URL 中的参数来动态决定使用哪个动态拓展点实例.

## 2	@Adaptive 注解

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Adaptive {
    String[] value() default {};
}
```

`@Adaptive` 为拓展加载器提供拓展点实例注入的信息, 可以使用在类和方法上, 如果使用在类上, 说明该类是一个手工实现的适配器类, 标记在方法上则会引导 Dubbo 为该方法自动生成适配代码, 其接收一个 `String[]` 用于指定多个拓展点名称, 如: `@Adaptive({"a", "b", "c"})`, 尝试顺序为 `c → b → a → 默认值`, 如果没有置顶任何参数, 则使用接口名称驼峰转 `.` 分隔符命名作为 "c".

## 3	自适应拓展原理

核心方法: `org.apache.dubbo.common.extension.ExtensionLoader#getAdaptiveExtension`

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
					// 调用 createAdaptiveExtension() 方法创建自适应实例
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

检查缓存, 若不存在已创建好的自适应实例, 则调用 `createAdaptiveExtension()` 方法创建:

```java
// org.apache.dubbo.common.extension.ExtensionLoader#getAdaptiveExtensionClass
// 创建自适应扩展点方法
private T createAdaptiveExtension() {
    try {
        // 这一行从 newInstance 这个关键字便知道这行代码就是创建扩展点的核心代码
        T instance = (T) getAdaptiveExtensionClass().newInstance();
        
        // 针对创建出来的实例对象做的一些类似 Spring 的前置后置的方式处理
        instance = postProcessBeforeInitialization(instance, null);
        instance = injectExtension(instance);
        instance = postProcessAfterInitialization(instance, null);
        initExtension(instance);
        return instance;
    } catch (Exception e) {
        throw new IllegalStateException("Can't create adaptive extension " + type + ", cause: " + e.getMessage(), e);
    }
}
```

```java
// org.apache.dubbo.common.extension.ExtensionLoader#getAdaptiveExtensionClass
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
```

```java
// 创建自适应扩展点类对象                  
private Class<?> createAdaptiveExtensionClass() {
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
    org.apache.dubbo.common.compiler.Compiler compiler = extensionDirector.getExtensionLoader(
        org.apache.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();
    // 通过调用 compile 方法，也就大致明白了，就是通过源代码生成一个类对象而已
    return compiler.compile(type, code, classLoader);
}
```

---