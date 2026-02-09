---
type: permanent
banner:
---
---

**关键词**: Dubbo, Remoting

---

Endpoint 抽象了 "**端点**" 概念, 可以认为 ip + port 唯一确定一个端点. 提供用于获取 Endpoint 属性的 get 系列方法, 可以获取到 Endpoint 的本地地址, 关联 URL, 以及 ChannelHandler, 提供 send 方法, 用于向该 Endpoint 发送数据; close 系列方法, 用于关闭一次连接, isClosed 检查连接是否已经关闭.

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

---