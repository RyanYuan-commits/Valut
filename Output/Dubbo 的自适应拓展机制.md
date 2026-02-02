---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
---
---

**关键词**: Dubbo, 自适应, 拓展 

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

### 1.1	@Adaptive 注解

**作用**: 为拓展加载器提供拓展点实例注入的信息.

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Adaptive {
    String[] value() default {};
}
```



---