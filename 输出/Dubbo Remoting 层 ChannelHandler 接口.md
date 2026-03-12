---
type: permanent
banner: Assets/Banner/pexels-8kspain-21564213.jpg
---
---

**关键词**: Dubbo, Remoting

---

ChannelHandler 是注册在 Channel 上的消息处理器, 可以用于处理 Channel 的连接建立以及连接断开事件, 还可以处理发送, 接收到的数据, 处理捕获到的异常; 方法的命名全部是过去式, 这些都是已经发生过的事件, ChannelHandler 是在这个事件发生过之后进行处理的.

```java
// org.apache.dubbo.remoting.Channel
public interface ChannelHandler {

    void connected(Channel channel) throws RemotingException;
	
    void disconnected(Channel channel) throws RemotingException;
	
    void sent(Channel channel, Object message) throws RemotingException;
	
    void received(Channel channel, Object message) throws RemotingException;
	
    void caught(Channel channel, Throwable exception) throws RemotingException;
    
}
```

---