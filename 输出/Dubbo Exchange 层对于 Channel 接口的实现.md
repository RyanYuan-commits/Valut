---
type: permanent
banner: Assets/Banner/pexels-eliannedipp-4666748.jpg
aliases:
  - ExchangeChannel
  - HeaderExchangeChannel
---
---

**关键词**: Dubbo, Exchange, Channel

---

## 1	ExchangeChannel 接口

继承了 Channel 接口, 并在此基础上抽象了 Exchange 层的**网络连接**; Dubbo 在 Exchange 层完成了**同步和异步**的转换, 上层只需要处理 `CompletableFuture` 即可.

```java
// org.apache.dubbo.remoting.exchange.ExchangeChannel
public interface ExchangeChannel extends Channel {
	
	// ...... 废弃接口 ......
	
    CompletableFuture<Object> request(Object request, ExecutorService executor) throws RemotingException;
	
    CompletableFuture<Object> request(Object request, int timeout, ExecutorService executor) throws RemotingException;
	
    ExchangeHandler getExchangeHandler();
	
    @Override
    void close(int timeout);
	
}
```

## 2	HeaderExchangeChannel

```java
// org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeChannel
final class HeaderExchangeChannel implements ExchangeChannel {  
	  
    private static final String CHANNEL_KEY = HeaderExchangeChannel.class.getName() + ".CHANNEL";  
	  
    private final Channel channel;  
	  
    private final int shutdownTimeout;  
	  
    private volatile boolean closed = false;
	
    // ......
}
```

其本质上是一个 `Channel` 实例的装饰器, 关键方法是委托给这个 `Channel` 实例完成的, 其实现的 `request()` 方法是 Dubbo 同步异步统一的关键步骤, 方法返回一个 `DefaultFuture` 实例.

```java
// org.apache.dubbo.remoting.exchange.support.header.HeaderExchangeChannel#request(java.lang.Object, int, java.util.concurrent.ExecutorService)
@Override  
public CompletableFuture<Object> request(Object request, int timeout, ExecutorService executor) throws RemotingException {  
    if (closed) {  
		// ... throw an exception ...
    }  

	// 构建 Request 对象
    Request req;  
    if (request instanceof Request) {  
        req = (Request) request;  
    } else {  
        // create request.  
        req = new Request();  
        req.setVersion(Version.getProtocolVersion());  
        req.setTwoWay(true);  
        req.setData(request);  
    }  

	// 返回值是一个 DefaultFuture	
    DefaultFuture future = DefaultFuture.newFuture(channel, req, timeout, executor);  
    try {  
        channel.send(req);  
    } catch (RemotingException e) {  
        future.cancel();  
        throw e;  
    }  
    return future;  
}
```

---