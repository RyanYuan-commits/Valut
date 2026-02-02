---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
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

### 1.1	@Adaptive 注解

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Adaptive {
    String[] value() default {};
}
```

`@Adaptive` 为拓展加载器提供拓展点实例注入的信息, 可以使用在类和方法上, 如果使用在类上, 说明该类是一个手工实现的适配器类, 标记在方法上则会引导 Dubbo 为该方法自动生成适配代码, 其接收一个 `String[]`.

```java
@SPI(value = "netty", scope = ExtensionScope.FRAMEWORK)
public interface Transporter {

    /**
     * Bind a server.
     *
     * @param url     server url
     * @param handler
     * @return server
     * @throws RemotingException
     * @see org.apache.dubbo.remoting.Transporters#bind(URL, ChannelHandler...)
     */
    @Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY})
    RemotingServer bind(URL url, ChannelHandler handler) throws RemotingException;

    /**
     * Connect to a server.
     *
     * @param url     server url
     * @param handler
     * @return client
     * @throws RemotingException
     * @see org.apache.dubbo.remoting.Transporters#connect(URL, ChannelHandler...)
     */
    @Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})
    Client connect(URL url, ChannelHandler handler) throws RemotingException;
}
```



---