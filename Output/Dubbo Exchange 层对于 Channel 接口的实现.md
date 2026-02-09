---
type: permanent
banner:
---
---

**关键词**: Dubbo, Exchange, Channel

---

## 1	ExchangeChannel 接口

继承了 Channel 接口, 并在此基础上抽象了 Exchange 层的**网络连接**; 

Dubbo 在 Exchane 层完成了同步和异步的转换, request 方法的两个重载返回值均为 CompletableFuture. 

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

---