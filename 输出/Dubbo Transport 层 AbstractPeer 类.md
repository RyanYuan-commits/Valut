---
type: permanent
banner: Assets/Banner/pexels-ken-cheung-3355734-5574638.jpg
aliases:
  - AbstractPeer
---
---

**关键词**: Dubbo, Transport

---

## 1	继承关系

`AbstractPeer` 同时实现了 `Endpoint` 和 `ChannelHandler` 接口, 表明其是一个无状态的网络通信端点, 并且具有处理通信中发生事件的能力. AbstractChannel 和 AbstractEndpoint 均为 AbstractPeer 的子类. 

```java
public abstract class AbstractPeer implements Endpoint, ChannelHandler { }
```

![[Dubbo AbstractPeer 类层次结构.png|600]]

## 2	关键属性

```java
// org.apache.dubbo.remoting.transport.AbstractPeer
public abstract class AbstractPeer implements Endpoint, ChannelHandler {  
  
    private final ChannelHandler handler;  

	// 端点自身状态
    private volatile URL url;  
  
    // closing closed means the process is being closed and close is finished  
    private volatile boolean closing;  
  
    private volatile boolean closed;
	
	// ......
	
}
```

表示端点自身状态的 URL 字段, 两个 Boolean 类型字段, closing 和 closed, 用于记录端点的状态, 这三个字段均与 Endpoint 接口相关; 还有一个字段指向了 `ChannelHandler` 实例, `AbstractPeer` 对于 `ChannelHandler` 接口的实现都委托给了这个 `ChannelHandler` 对象, 这个结论同样适用于其子类, `AbstractChannel`. `AbstractServer`, `AbstractClient`, 它们都需要关联一个 `ChannelHandler` 实例.

---