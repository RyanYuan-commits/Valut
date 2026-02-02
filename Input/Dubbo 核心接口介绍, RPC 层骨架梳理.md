---
source:
type: input
---
# 📰 阅读笔记

Protocol 层是 Remoting 层的使用者, 会通过 Exchangers 门面类创建 ExchangeClient 和 ExchangeServer, 还会创建相应的 ChannelHandler 实现和 Codec2 实现并交给 Exchange 层进行装饰.

![[Dubbo Protocol 层.png|800]]

## 1	模块结构

Protocol 层在源码中对应的是 dubbo-rpc 模块, 这个模块的结构为:

```
dubbo-rpc
-	dubbo-rpc-api
-	dubbo-rpc-dubbo
-	dubbo-rpc-injvm
-	dubbo-rpc-triple
```

与 dubbo-remoting 模块类型, 其中 dubbo-rpc-api 是对具体协议, 服务暴露, 服务引用, 代理等抽象, 是整个 Protocol 的核心. 剩余的模块是 Dubbo 对具体支持协议的实现.

```
.
|-- aot
|-- cluster
|-- filter
|-- listener
|-- protocol
|-- proxy
|-- stub
`-- support
```

- filter: 在服务引用时会进行一些列的过滤, 内部包含了大量的过滤器;
	
- listener: 在服务发布和服务引用过程中, 可以添加一些 Listener 来监听相应的时间, 与 Listener 相关的接口 Adapter, Wrapper 实现就在这个包内;
	
- protocol: 一些实现了 Protocol 接口以及 Invoker 接口的抽象类在这个包中, 他们主要是为了 Protocol 和 Invoker 接口的具体实现提供一些公共逻辑;
	
- proxy: 提供了创建代理的能力, 支持了 JDK 动态代理以及 Javaassist 字节码两种方式生成本地代理类;
	
- support: 包含了 RpcUtils 工具类, Mock 相关 Protocol 实现以及 Invoker 实现.

## 2	核心接口

### 2.1	Invoker

Invoker 接口渗透在整个 Dubbo 代码实现中, Dubbo 中很多的设计思路都会向 Invoker 这个概念靠拢.

![[Dubbo Invoker.png|700]]

有两种最为关键的 Invoker, Provider 的 Invoker  和 Consumer 的 Invoker:

- Consumer 拿到一个 Proxy 后, 调用对端的逻辑是通过 Invoker 实现的
	
- Provider 端对某个接口的实现类会被封装成一个 AbstractProxyInvoker 实例, 并生成对应的 Exporter 实例, 当 Dubbo Protocol 层收到一个请求后, 会找到这个 Exporter 实例, 并调用其对应的 AbstractProxyInvoker 实例, 从而完成 Provider 逻辑的调用.

```java
public interface Invoker<T> extends Node {

	// 服务接口
    Class<T> getInterface();

	// 进行一次调用
    Result invoke(Invocation invocation) throws RpcException;
}
```

### 2.2	Invocation

Invocation 是 invoke 方法的入参, 抽象了一次 RPC 调用的目标服务和方法信息, 相关参数信息, 具体的参数值以及一些附加信息:

```java
public interface Invocation {

    // 调用Service的唯一标识
    String getTargetServiceUniqueName();

    // 调用的方法名称
    String getMethodName();

    // 调用的服务名称
    String getServiceName();

    // 参数类型集合
    Class<?>[] getParameterTypes();

    // 参数签名集合
    default String[] getCompatibleParamSignatures() {
        return Stream.of(getParameterTypes())
                .map(Class::getName)
                .toArray(String[]::new);
    }

    // 此次调用具体的参数值
    Object[] getArguments();

    // 此次调用关联的Invoker对象
    Invoker<?> getInvoker();

    // Invoker对象可以设置一些KV属性，这些属性并不会传递给Provider
    Object put(Object key, Object value);

    Object get(Object key);

    Map<Object, Object> getAttributes();

    // Invocation可以携带一个KV信息作为附加信息，一并传递给Provider，
    // 注意与 attribute 的区分
    Map<String, String> getAttachments();

    Map<String, Object> getObjectAttachments();

    void setAttachment(String key, String value);

    void setAttachment(String key, Object value);

    void setObjectAttachment(String key, Object value);

    void setAttachmentIfAbsent(String key, String value);

    void setAttachmentIfAbsent(String key, Object value);

    void setObjectAttachmentIfAbsent(String key, Object value);

    String getAttachment(String key);

    Object getObjectAttachment(String key);

    String getAttachment(String key, String defaultValue);

    Object getObjectAttachment(String key, Object defaultValue);
}
```

### 2.3	Result 接口

Result 接口是 invoke 方法的返回值, 抽象了一次调用的返回值, 其中包含了被调用方返回值 (或异常) 以及附加信息, 我们也可以添加回调方法, 在 RPC 调用方法结束后触发这些回调.

```java
public interface Result extends Serializable {

    // 获取/设置此次调用的返回值
    Object getValue();

    void setValue(Object value);

    // 如果此次调用发生异常，则可以通过下面三个方法获取
    Throwable getException();

    void setException(Throwable t);

    boolean hasException();

    // recreate()方法是一个复合操作，如果此次调用发生异常，则直接抛出异常，
    // 如果没有异常，则返回结果
    Object recreate() throws Throwable;

    // 添加一个回调，当RPC调用完成时，会触发这里添加的回调
    Result whenCompleteWithContext(BiConsumer<Result, Throwable> fn);

    <U> CompletableFuture<U> thenApply(Function<Result, ? extends U> fn);

    // 阻塞线程，等待此次RPC调用完成(或是超时)
    Result get() throws InterruptedException, ExecutionException;

    Result get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;

    // Result中同样可以携带附加信息
    Map<String, String> getAttachments();

    Map<String, Object> getObjectAttachments();

    void addAttachments(Map<String, String> map);

    void addObjectAttachments(Map<String, Object> map);

    void setAttachments(Map<String, String> map);

    void setObjectAttachments(Map<String, Object> map);

    String getAttachment(String key);

    Object getObjectAttachment(String key);

    String getAttachment(String key, String defaultValue);

    Object getObjectAttachment(String key, Object defaultValue);

    void setAttachment(String key, String value);

    void setAttachment(String key, Object value);

    void setObjectAttachment(String key, Object valu

}
```

### 2.4	Exporter 接口

Exporter 暴露 Invoker 的实现, 让 Provider 根据请求的各种信息. 找到对应的 Invoker.

```java
public interface Exporter<T> {

    /**
     * get invoker.
     *
     * @return invoker
     */
    Invoker<T> getInvoker();

    /**
     * unexport.
     * <p>
     * <code>
     * getInvoker().destroy();
     * </code>
     */
    void unexport();

    /**
     * register to registry
     */
    void register();

    /**
     * unregister from registry
     */
    void unregister();
	
}
```

### 2.5	ExporterListener

为了监听服务的发布事件以及取消暴露事件, Dubbo 定义了一个 SPI 拓展接口, ExporterListener, 其定义如下:

```java
@SPI(scope = ExtensionScope.FRAMEWORK)
public interface ExporterListener {

    void exported(Exporter<?> exporter) throws RpcException;

    void unexported(Exporter<?> exporter);

}
```

分别提供了服务发布时和服务撤销时会调用的拓展方法.

### 2.6	InvokerListener

相应的, Dubbo 还提供了对于 Invoker 的监听器, 可以监听 Consumer 引用服务时触发的事件:

```java
@SPI
public interface InvokerListener {

    void referred(Invoker<?> invoker) throws RpcException;

    void destroyed(Invoker<?> invoker);
}
```

### 2.7	Protocol 接口

核心接口, 定义了 export 和 refer 两个核心方法.

```java
// 默认使用 Dubbo 协议
@SPI(value = "dubbo", scope = ExtensionScope.FRAMEWORK)
public interface Protocol {

	// 获取默认端口
    int getDefaultPort();

	// 将一个 Invoker 暴露出去, export 方法需要实现幂等, 服务暴露一次和多次效果是相同的
    @Adaptive
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;

	// 应用一个 Invoker, 根据参数返回一个 Invoker 对象, Consumer 可以使用这个 Invoker 请求到 Provider 端
    @Adaptive
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;

	// 销毁 export 或者 refer 使用到的 Invoker 对象, 释放当前 Protocol 对象底层占用的资源
    void destroy();

	// 返回当前 Protocol 底层全部的 ProtocolServer
    default List<ProtocolServer> getServers() {
        return Collections.emptyList();
    }
	
}
```

在 Protocol 的实现中, export 底层还涉及代理对象的创建, 底层 Server 的启动等操作, refer 方法还设计 Client 的创建等操作.

### 2.8	ProxyFactory

Protocol 层, 创建代理对象的工厂, 是一个拓展接口, getProxy 方法为 Invoker 创建代理对象, getInvoker 将代理对象反向封装成 Invoker 对象:

```java
@SPI(value = "javassist", scope = FRAMEWORK)
public interface ProxyFactory {

    @Adaptive({PROXY_KEY})
    <T> T getProxy(Invoker<T> invoker) throws RpcException;

    @Adaptive({PROXY_KEY})
    <T> T getProxy(Invoker<T> invoker, boolean generic) throws RpcException;

    @Adaptive({PROXY_KEY})
    <T> Invoker<T> getInvoker(T proxy, Class<T> type, URL url) throws RpcException;
	
}
```

### 2.9	Filter 接口

```java
@SPI(scope = ExtensionScope.MODULE)
public interface Filter extends BaseFilter {}

public interface BaseFilter {

	// 将请求传递给后续的 Invoker 处理
    Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException;

	// 用户监听响应机器异常
    interface Listener {

        void onResponse(Result appResponse, Invoker<?> invoker, Invocation invocation);

        void onError(Throwable t, Invoker<?> invoker, Invocation invocation);
    }
}
```

Filter 也是一个可拓展接口, Dubbo 提供丰富的 Filter 实现来进行功能拓展.

## 3	服务暴露全流程

Protocol 的继承关系:

```java
Protocol (org.apache.dubbo.rpc)
	AbstractProtocol (org.apache.dubbo.rpc.protocol)
	    DubboProtocol (org.apache.dubbo.rpc.protocol.dubbo)
	    TripleProtocol (org.apache.dubbo.rpc.protocol.tri)
	    AbstractProxyProtocol (org.apache.dubbo.rpc.protocol)
	    InjvmProtocol (org.apache.dubbo.rpc.protocol.injvm)
	    MockProtocol (org.apache.dubbo.rpc.support)
	ProtocolSecurityWrapper (org.apache.dubbo.rpc.protocol)
	InvokerCountWrapper (org.apache.dubbo.rpc.protocol)
	ProtocolListenerWrapper (org.apache.dubbo.rpc.protocol)
	QosProtocolWrapper (org.apache.dubbo.qos.protocol)
	ProtocolSerializationWrapper (org.apache.dubbo.rpc.protocol)
	ProtocolFilterWrapper (org.apache.dubbo.rpc.cluster.filter)
```

### 3.1	AbstractProtocol

AbstractProtocol 提供了一些 Protocol 实现需要的公共能力以及公共字段, 核心字段有:

- exporterMap: 为 Map 类型, key 为服务的 uri, value 为具体的 Exporter;
	
- serverMap: 记录了所有 ProtocolServer 实例, 其中的 key 是 host  和 port  组成的字符串 ,value 是监听该地址的 ProtocolServer, ProtocolServer 是对 RemotingServer 的简单封装, 表示一个服务端;
	
- invokers: 服务引用的集合.

AbstractProtocol 仅实现 destory 方法, 销毁所有引入的 invoker 和发布出去的服务:

```java
public void destroy() {
    for (Invoker<?> invoker : invokers) {
        if (invoker != null) {
            invokers.remove(invoker);
            invoker.destroy(); // 关闭全部的服务引用
        }
    }
    for (String key : new ArrayList<String>(exporterMap.keySet())) {
        Exporter<?> exporter = exporterMap.remove(key);
        if (exporter != null) {
            exporter.unexport(); // 关闭暴露出去的服务
        }
    }
}
```

### 3.2	DubboExporter.export

DubboProtocol 对 export 方法的实现.

```java
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    URL url = invoker.getUrl();
    // 创建ServiceKey，其核心实现在前文已经详细分析过了，这里不再重复
    String key = serviceKey(url); 
    // 将上层传入的Invoker对象封装成DubboExporter对象，然后记录到exporterMap集合中
    DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
    exporterMap.put(key, exporter);
    ... // 省略一些日志操作
    // 启动ProtocolServer
    openServer(url);
    // 进行序列化的优化处理
    optimizeSerialization(url);
    return exporter;
}
```

DubboExporter 对 Invoker 的封装:

AbstractExporter 中维护了一个 Invoker 对象，以及一个 unexported 字段（boolean 类型），在 unexport() 方法中会设置 unexported 字段为 true，并调用 Invoker 对象的 destory() 方法进行销毁。

DubboExporter 也比较简单，其中会维护底层 Invoker 对应的 ServiceKey 以及 DubboProtocol 中的 exportMap 集合，在其 unexport() 方法中除了会调用父类 AbstractExporter 的 unexport() 方法之外，还会清理该 DubboExporter 实例在 exportMap 中相应的元素。

```java
DubboProtocol.export(Invoker<T>) (org.apache.dubbo.rpc.protocol.dubbo)
	DubboProtocol.openServer(URL) (org.apache.dubbo.rpc.protocol.dubbo)
		DubboProtocol.createServer(URL) (org.apache.dubbo.rpc.protocol.dubbo)
			Exchangers.bind(URL, ExchangeHandler) (org.apache.dubbo.remoting.exchange)
				HeaderExchange.bind(URL, ExchangeHandler) (org.apache.dubbo.remoting.exchange.support.header)
					Transporters.bind(URL, ChannelHandler) (org.apache.dubbo.remoting)
						NettyTransporter.bind(URL, ChannelHandler) (org.apache.dubbo.remoting.transport.netty4)
							NettyServer(URL, ChannelHandler) (org.apache.dubbo.remoting.transport.netty4)
```

### 3.3	DubboProtocol.openServer

在 DubboExporter.export 方法中, 会调用 openServer 方法, openServer 一路调用 Exchange 层, Transport 层, 并最终创建 NettyServer 来接受客户端的请求.

```java
private void openServer(URL url) {
    String key = url.getAddress(); // 获取host:port这个地址
    boolean isServer = url.getParameter(IS_SERVER_KEY, true);
    if (isServer) { // 只有Server端才能启动Server对象
        ProtocolServer server = serverMap.get(key);
        if (server == null) { // 无ProtocolServer监听该地址
            synchronized (this) { // DoubleCheck，防止并发问题
                server = serverMap.get(key);
                if (server == null) {
                    // 调用createServer()方法创建ProtocolServer对象
                    serverMap.put(key, createServer(url));
                }
            }
        } else { 
            // 如果已有ProtocolServer实例，则尝试根据URL信息重置ProtocolServer
            server.reset(url);
        }
    }
}
```

### 3.4	DubboProtocol.createServer

```java
private ProtocolServer createServer(URL url) {
    url = URLBuilder.from(url)
            // ReadOnly请求是否阻塞等待
            .addParameterIfAbsent(CHANNEL_READONLYEVENT_SENT_KEY, Boolean.TRUE.toString())
            // 心跳间隔
            .addParameterIfAbsent(HEARTBEAT_KEY, String.valueOf(DEFAULT_HEARTBEAT))
            .addParameter(CODEC_KEY, DubboCodec.NAME) // Codec2扩展实现
            .build();
    // 检测SERVER_KEY参数指定的Transporter扩展实现是否合法
    String str = url.getParameter(SERVER_KEY, DEFAULT_REMOTING_SERVER); 
    if (str != null && str.length() > 0 && !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str)) {
        throw new RpcException("...");
    }
    // 通过Exchangers门面类，创建ExchangeServer对象
    ExchangeServer server = Exchangers.bind(url, requestHandler);
	
    ... // 检测CLIENT_KEY参数指定的Transporter扩展实现是否合法(略)
    // 将ExchangeServer封装成DubboProtocolServer返回
    return new DubboProtocolServer(server);
}
```

在 createServer 方法中, 首先会为 URL 添加一些默认值, 同时会进行一些参数的检测, 主要有五个:

- HEARTBEAT_KEY 参数值, 默认值为 60000, 表示默认的心跳时间间隔为 60 秒. 
	
- CHANNEL_READONLYEVENT_SENT_KEY 参数值, 默认值为 true, 表示 ReadOnly 请求需要阻塞等待响应返回. 在 Server 关闭的时候, 只能发送 ReadOnly 请求, 这些 ReadOnly 请求由这里设置的 CHANNEL_READONLYEVENT_SENT_KEY 参数值决定是否需要等待响应返回. 
	
- CODEC_KEY 参数值, 默认值为 dubbo. 你可以回顾 Codec2 接口中 @Adaptive 注解的参数, 都是获取该 URL 中的 CODEC_KEY 参数值. 
	
- 检测 SERVER_KEY 参数指定的扩展实现名称是否合法, 默认值为 netty. 你可以回顾 Transporter 接口中 @Adaptive 注解的参数, 它决定了 Transport 层使用的网络库实现, 默认使用 Netty 4 实现. 
	
- 检测 CLIENT_KEY 参数指定的扩展实现名称是否合法. 同 SERVER_KEY 参数的检查流程. 

完成上述参数的设置后, 就可以通过 Exchange 门面类创建 ExchangeServer, 并封装成 DubboProtocolServer 返回.

### 3.5	DubboCountCodec

在创建 ExchangeServer 时, 使用的 Codec2 接口实际上是 DubboCountCodec, 配置在对应的 SPI 文件中.

DubboCountCodec 中维护了一个 DubboCodec 对象, 其解码能力是这个对象提供的, DubboCountCodec 只负责在解码过程中 ChannelBuffer 的 readIndex 的指针控制.

DubboCodec 是 ExchangeCodec 的子类, ExchangeCodec 只处理了 Dubbo 的协议请求头, DubboCodec 在此基础上, 添加了解析 Dubbo 消息提的能力.

DubboCodec 覆盖 ExchangeCodec  的 encodeRequestData 方法, 按照 Dubbo 协议的格式编码.

### 3.6	RpcInvocation

RpcInvocation 实现了 Invocation 接口, 核心字段如下, 通过读写这些字段就实现了 Invocation 接口的全部方法:

- targetServiceUniqueName (String类型): 要调用的唯一服务名称, 其实就是 ServiceKey, 即 interface/group:version 三部分构成的字符串. 
	
- methodName (String类型): 调用的目标方法名称. 
	
- serviceName (String类型): 调用的目标服务名称, 示例中就是org.apache.dubbo.demo.DemoService. 
	
- parameterTypes (Class 数组类型): 记录了目标方法的全部参数类型. 
	
- parameterTypesDesc (String类型): 参数列表签名. 
	
- arguments (Object 数组类型): 具体参数值. 
	
- attachments (Map类型): 此次调用的附加信息, 可以被序列化到请求中. 
	
- attributes (Map类型): 此次调用的属性信息, 这些信息不能被发送出去. 
	
- invoker (Invoker类型): 此次调用关联的 Invoker 对象. 
	
- returnType (Class 类型): 返回值的类型. 
	
- invokeMode (InvokeMode类型): 此次调用的模式, 分为 SYNC、ASYNC 和 FUTURE 三类. 

RpcInvocation 的子类是 DecodeableRpcInvocation, 是用来支持解码的, 其实现的 decode 方法正好是 DubboCodec.encodeRequestData 方法对应的解码操作, 在 DubboCodec.decodeBody 方法就调用了这个方法.

Dubbo 线程模型: AllDispatcher 实现为例给出结论. 

- IO 线程内执行的 ChannelHandler 实现依次有: InternalEncoder、InternalDecoder(两者底层都是调用 DubboCodec)、IdleStateHandler、MultiMessageHandler、HeartbeatHandler 和 NettyServerHandler. 
- 在非 IO 线程内执行的 ChannelHandler 实现依次有: DecodeHandler、HeaderExchangeHandler 和 DubboProtocol$requestHandler. 

在 DubboProtocol 中有一个 requestHandler 字段, 是一个实现了 ExchangeHandlerAdapter 的实例, 间接实现了 ExchangeHandler 接口, 核心方法为:

```java
public CompletableFuture<Object> reply(ExchangeChannel channel, Object message) throws RemotingException {
    ... // 这里省略了检查message类型的逻辑，通过前面Handler的处理，这里收到的message必须是Invocation类型的对象
    Invocation inv = (Invocation) message;
    // 获取此次调用Invoker对象
    Invoker<?> invoker = getInvoker(channel, inv);
    ... // 针对客户端回调的内容，在后面详细介绍，这里不再展开分析
    // 将客户端的地址记录到RpcContext中
    RpcContext.getContext().setRemoteAddress(channel.getRemoteAddress());
    // 执行真正的调用
    Result result = invoker.invoke(inv);
    // 返回结果
    return result.thenApply(Function.identity());
}
```

getInvoker 方法会先根据 Invocation 携带的信息构造 ServiceKey, 然后从 exporterMap 中查找对应的 DubboExporter 对象, 最终获取底层的 Invoker 对象.

```java
Invoker<?> getInvoker(Channel channel, Invocation inv) throws RemotingException {
    ... // 省略对客户端Callback以及stub的处理逻辑，后面单独介绍
    String serviceKey = serviceKey(port, path, (String) inv.getObjectAttachments().get(VERSION_KEY),
            (String) inv.getObjectAttachments().get(GROUP_KEY));
    DubboExporter<?> exporter = (DubboExporter<?>) exporterMap.get(serviceKey);
    ... //  查找不到相应的DubboExporter对象时，会直接抛出异常，这里省略了这个检测
    return exporter.getInvoker(); // 获取exporter中获取Invoker对象
}
```

## 4	序列化优化处理

下面我们回到 DubboProtocol.export 方法继续分析, 在完成 ProtocolServer 的启动之后, export 方法最后会调用 optimizeSerialization 方法对指定的序列化算法进行优化. 

这里先介绍一个基础知识, 在使用某些序列化算法 (例如,  Kryo、FST 等)时, 为了让其能发挥出最佳的性能, 最好将那些需要被序列化的类提前注册到 Dubbo 系统中. 例如, 我们可以通过一个实现了 SerializationOptimizer 接口的优化器, 并在配置中指定该优化器, 如下示例代码: 

```java
public class SerializationOptimizerImpl implements SerializationOptimizer {
    public Collection<Class> getSerializableClasses() {
        List<Class> classes = new ArrayList<>();
        classes.add(xxxx.class); // 添加需要被序列化的类
        return classes;
    }
}
```

在 DubboProtocol.optimizeSerialization 方法中, 就会获取该优化器中注册的类, 通知底层的序列化算法进行优化, 序列化的性能将会被大大提升. 当然, 在进行序列化的时候, 难免会级联到很多 Java 内部的类 (例如, 数组、各种集合类型等), Kryo、FST 等序列化算法已经自动将JDK 中的常用类进行了注册, 所以无须重复注册它们. 

下面我们回头来看 optimizeSerialization() 方法, 分析序列化优化操作的具体实现细节: 

```java
private void optimizeSerialization(URL url) throws RpcException {
    // 根据URL中的optimizer参数值, 确定SerializationOptimizer接口的实现类
    String className = url.getParameter(OPTIMIZER_KEY, "");
    Class clazz = Thread.currentThread().getContextClassLoader().loadClass(className);
    // 创建SerializationOptimizer实现类的对象
    SerializationOptimizer optimizer = (SerializationOptimizer) clazz.newInstance();
    // 调用getSerializableClasses()方法获取需要注册的类
    for (Class c : optimizer.getSerializableClasses()) {
        SerializableClassRegistry.registerClass(c); 
    }
    optimizers.add(className);
}
```

SerializableClassRegistry 底层维护了一个 static 的 Map (REGISTRATIONS 字段), registerClass() 方法就是将待优化的类写入该集合中暂存, 在使用 Kryo、FST 等序列化算法时, 会读取该集合中的类, 完成注册操作


---

# 💭 我的思考

这个观点如何与我已知的知识产生联系? 它让我想到了什么?