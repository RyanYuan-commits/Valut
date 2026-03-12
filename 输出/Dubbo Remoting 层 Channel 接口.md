---
type: permanent
banner: Assets/Banner/pexels-faikackmerd-1025469.jpg
---
---

**关键词**: Dubbo, Remoting

---

**当连接建立后**, 通信双方会持有对方的 Channel, Dubbo 的 Channel 与 Netty Channel 的概念基本一致, 其具备 KV 属性存储的能力, 同时, 其继承 Endpoint 接口, 具备获取关闭状态, 发送数据的能力,

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

---