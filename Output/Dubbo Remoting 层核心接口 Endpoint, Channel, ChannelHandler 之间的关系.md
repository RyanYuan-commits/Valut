---
icon: 🦈
type: permanent
created: 2025-11-22
banner: "![[Pasted image 20251122105012.png]]"
---
# 🌐 核心观点

Endpoint, Channel, ChannelHandler 是 Dubbo Remoting 层重要的接口, 它们分别解决了通信中 "去哪里", "如何传输" 和 "做什么" 的问题.

# 🔖 详细解释

## 1	Endpoint

Endpoint 抽象了 "端点" 概念, 可以认为 ip + port 唯一确定一个端点.

```java
// org.apache.dubbo.remoting.Endpoint
public interface Endpoint {

    URL getUrl();

    ChannelHandler getChannelHandler();

    InetSocketAddress getLocalAddress();

    void send(Object message) throws RemotingException;

    void send(Object message, boolean sent) throws RemotingException;

    void close();

    void close(int timeout);

    void startClose();

    boolean isClosed();
    
}
```

- 提供用于获取 Endpoint 属性的 get 系列方法, 可以获取到 Endpoint 的本地地址, 关联 URL, 以及 ChannelHandler;
	
- 提供 send 方法, 用于向该 Endpoint 发送数据;
	
-  close 系列方法, 用于关闭一次连接, isClosed 检查连接是否已经关闭.

## 2	Channel

当连接建立后, 通信双方会持有对方的 Channel, Dubbo 的 Channel 与 Netty Channel 的概念基本一致.

```java
// 
public interface Channel extends Endpoint {

    InetSocketAddress getRemoteAddress();

    boolean isConnected();

    boolean hasAttribute(String key);

    Object getAttribute(String key);

    void setAttribute(String key, Object value);

    void removeAttribute(String key);
    
}
```

- Channel 具备存储 KV 属性的能力;
	
- 同时, 其继承 Endpoint 接口, 具备获取关闭状态, 以及发送数据的能力.

## 3	ChannelHandler

ChannelHandler 是注册在 Channel 上的消息处理器, 可以用于处理 Channel 的连接建立以及连接断开事件, 还可以处理发送, 接收到的数据, 处理捕获到的异常;

```java
// org.apache.dubbo.remoting.Channel
@SPI(scope = ExtensionScope.FRAMEWORK)
public interface ChannelHandler {

    void connected(Channel channel) throws RemotingException;

    void disconnected(Channel channel) throws RemotingException;

    void sent(Channel channel, Object message) throws RemotingException;

    void received(Channel channel, Object message) throws RemotingException;

    void caught(Channel channel, Throwable exception) throws RemotingException;
    
}
```

方法的命名全部是过去式, 这些都是已经发生过的事件, ChannelHandler 是在这个事件发生过之后进行处理的.

## 4	核心接口之间的关系

Endpoint 代表网络中的某一个端点, 它是无状态的, 回答了 "去哪里通信?" 的问题;

与 Endpoint 不同, Channel 是有状态的, 当两个端点之间建立起了连接, 双方会持有对方的一个 Channel, 是传输数据的动态管道, Channel 伴随连接一起诞生;

ChannelHandler 与 Channel 绑定, 处理具体的业务逻辑以及协议逻辑, 回答了 "通信中可以做什么?" 的问题.

---

# 📚 参考内容

- Dubbo 官方文档 - 代码架构: https://dubbo.apache.org/zh-cn/overview/mannual/java-sdk/reference-manual/architecture/code-architecture/