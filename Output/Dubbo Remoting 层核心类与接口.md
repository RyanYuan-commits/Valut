---
created: 2025-11-24
type: permanent
banner: Assets/Banner/pexels-jeremy-bishop-1260133-2524874.jpg
---
# 🌐 核心观点

Remoting 层提供了多种客户端和服务端通信的能力, 包含了 Exchange, Transport 和 Serialize 三个层级.

---

# 🔖 详细解释

## 1	Remoting 层总览

### 1.1	 子层级架构

![[Dubbo Remoting 层.png|900]]

Dubbo Remoting 层提供了**客户端与服务端的通信**功能, 包含了三个子层级:

- **Exchange 层**: 封装请求响应模式, 同步转异步, 以 `Request`, `Response` 为中心;
	
- **Transport 层**: 抽象 mina 和 netty 为统一接口, 以 `Message` 为中心;
	
- **Serialize 层**: 可复用的一些工具.

### 1.2	核心包: dubbo-remoting-api

dubbo-remoting 模块中的 dubbo-remoting-api 是其他模块的顶层抽象, 有这些关键包:

- buffer 包: 定义了缓冲区相关的接口, 抽象类和实现类, 缓冲区是 NIO 框架不可获取的角色, 在各个 NIO 框架中都有自己的缓冲区实现, 这里的缓冲区是更高层面的抽象, 抽象了各个 NIO 框架的缓冲区, 同时也提供了一些基础实现;
	
- exchange 包: 抽象了 Request 和 Reponse 两个概念, 并为其添加了很多特性, 是整个远程调用中非常核心的部分;
	
- transport 包: 对于网络传输层的抽象,但它只负责抽象单向的消息传输, 即请求从 Client 端发出, Server 端接收; 响应消息从 Server 端发出, Client 端发出, 有很多网络库可以实现网络传输的功能, 例如 Netty, Grizzly 等, transport 包是在这些网络库上层的一层抽象;
	
- 其他接口: Endpoint, Channel, Transport, Dispatcher 等顶层接口也在这个包中, 它们是 Dubbo Remoting 的核心接口.

## 2	Endpoint, Channel, ChannelHandler

详见: [[Dubbo Remoting 层核心接口 Endpoint, Channel, ChannelHandler 之间的关系]]

## 3	Codec2

```java
@SPI(scope = ExtensionScope.FRAMEWORK)
public interface Codec2 {

    @Adaptive({Constants.CODEC_KEY})
    void encode(Channel channel, ChannelBuffer buffer, Object message) throws IOException;

    @Adaptive({Constants.CODEC_KEY})
    Object decode(Channel channel, ChannelBuffer buffer) throws IOException;

    enum DecodeResult {
    
        NEED_MORE_INPUT, SKIP_SOME_INPUT
        
    }
    
}
```

Netty 中, 有一类专门负责实现编解码功能的 ChannelHandler, Dubbo 的 Codec 接口就是对类似功能的的抽象.

## 4	Client & RemotingServer

Client 和 RemotingServer 接口分别抽象了通信中客户端和服务端的能力, 两者都继承了 Channel 接口, 说明它们都是有状态的接口.

![[Dubbo Client and RemotingServer.png|800]]

Client 只能关联一个 Channel, 而 RemotingServer 可以接收 Client 发起的多个 Channel, 所以在 RemotingServer 接口中定义了查询 Channel 的相关方法.

```java
// org.apache.dubbo.remoting.RemotingServer
public interface RemotingServer extends Endpoint, Resetable, IdleSensible {

    boolean isBound();

    Collection<Channel> getChannels();

    Channel getChannel(InetSocketAddress remoteAddress);

    @Deprecated
    void reset(org.apache.dubbo.common.Parameters parameters);
    
}

// org.apache.dubbo.remoting.Client
public interface Client extends Endpoint, Channel, Resetable, IdleSensible {  
  
	void reconnect() throws RemotingException;  
  
    @Deprecated  
    void reset(org.apache.dubbo.common.Parameters parameters);  
  
}
```

## 5	Transporter

### 5.1	工厂接口 

Dubbo 定义了 Transporter 工厂接口, 用来创建 Client 和 RemotingServer 实例:

```java
@SPI(value = "netty", scope = ExtensionScope.FRAMEWORK)
public interface Transporter {

    @Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY})
    RemotingServer bind(URL url, ChannelHandler handler) throws RemotingException;

    @Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})
    Client connect(URL url, ChannelHandler handler) throws RemotingException;
    
}
```

对于每一个 Dubbo 支持的 NIO 库, 都有一个 Transporter  接口的实现类, 比如 NettyTransporter, MinaTransporter, GrizzlyTransporter 等.

### 5.2	门面类: Transports

Dubbo 还提供了一个门面类 Transports 来减轻外部使用 Transporter 接口的难度.

```java
// org.apache.dubbo.remoting.Transporters#getTransporter
public static Transporter getTransporter(URL url) {
	return url.getOrDefaultFrameworkModel()
			.getExtensionLoader(Transporter.class)
			.getAdaptiveExtension();
}
```

Transport 是一个可拓展接口, Transports 中通过 Dubbo SPI 根据 URL 参数获取其具体实现.

```java
// org.apache.dubbo.remoting.Transporters#bind
public static RemotingServer bind(URL url, ChannelHandler... handlers) throws RemotingException {
	if (url == null) {
		throw new IllegalArgumentException("url == null");
	}
	if (handlers == null || handlers.length == 0) {
		throw new IllegalArgumentException("handlers == null");
	}
	ChannelHandler handler;
	if (handlers.length == 1) {
		handler = handlers[0];
	} else {
		handler = new ChannelHandlerDispatcher(handlers);
	}
	return getTransporter(url).bind(url, handler);
}

// org.apache.dubbo.remoting.Transporters#connect
public static Client connect(URL url, ChannelHandler... handlers) throws RemotingException {  
    if (url == null) {  
        throw new IllegalArgumentException("url == null");  
    }  
    ChannelHandler handler;  
    if (handlers == null || handlers.length == 0) {  
        handler = new ChannelHandlerAdapter();  
    } else if (handlers.length == 1) {  
        handler = handlers[0];  
    } else {  
        handler = new ChannelHandlerDispatcher(handlers);  
    }  
    return getTransporter(url).connect(url, handler);  
}
```

在创建 Client 和 RemotingServer 的时候, 可以绑定多个 ChannelHandler, Transporters 使用这些 ChannelHandler 实例构建出 ChannelHandlerDispatcher 对象. 

ChannelHandlerDispatcher 实现了 ChannelHandler 接口, 其内部维护了一个 CopyOnWriteArraySet 集合, 当某个事件触发后, 会调用这个集合中所有元素的对应方法.

---

# 📚 参考内容

