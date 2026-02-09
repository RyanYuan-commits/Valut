---
type: permanent
banner: Assets/Banner/Pasted image 20251122105012.png
---
---

**关键词**: Dubbo, 异步, Exchange

---

## 1	关键字段

```java
// org.apache.dubbo.remoting.exchange.support.DefaultFuture
public class DefaultFuture extends CompletableFuture<Object> {  
  
    private static final ErrorTypeAwareLogger logger = LoggerFactory.getErrorTypeAwareLogger(DefaultFuture.class);  
	  
    private static final Map<Long, Channel> CHANNELS = new ConcurrentHashMap<>();  
	  
    private static final Map<Long, DefaultFuture> FUTURES = new ConcurrentHashMap<>();  
	  
    private static final GlobalResourceInitializer<Timer> TIME_OUT_TIMER = new GlobalResourceInitializer<>(  
            () -> new HashedWheelTimer(new NamedThreadFactory("dubbo-future-timeout", true), 30, TimeUnit.MILLISECONDS),  
            DefaultFuture::destroy);  
	  
    // 当前请求的 mId
    private final Long id;  
	
	// 发送请求的 Channel
    private final Channel channel;  
	
    // 当前请求的请求对象
    private final Request request;  
	
	// 整个请求-响应交互完成的超时时间. 整个请求-响应交互完成的超时时间. 
    private final int timeout;  
	
	// 创建时间
    private final long start = System.currentTimeMillis();  
	
	// 请求的发送时间
    private volatile long sent;  
	
	// 定时任务, 到期则说明对端响应超时, 执行超时处理逻辑
    private Timeout timeoutCheckTask;  
	
	// 请求关联的线程池
    private ExecutorService executor;
	
	// ......
	
}
```

`DefaultFuture` 继承 `CompletableFuture`, 在其中维护了两个静态的 `Map`:

- `CHNNELS`: 负责请求与 `Channel` 之间的关联关系, Key 为请求 ID, Value 为发送请求的 `Channel`;
	
- `FUTURES`: 负责请求与 `DefaultFuture` 之间的关联关系, Key 为请求 ID, Value 为请求对应的 `DefaultFuture`.

## 2	创建方法

```java
public static DefaultFuture newFuture(Channel channel, Request request, int timeout, ExecutorService executor) {  
    final DefaultFuture future = new DefaultFuture(channel, request, timeout);  
    future.setExecutor(executor);  
    // timeout check  
    timeoutCheck(future);  
    return future;  
}
```

使用静态方法 `newFuture()` 获取一个 `DefaultFuture` 实例, 这个方法会初始化一个 `DefaultFuture` 然后为其设置 `timeoutCheckTask`.

## 3	请求与响应流程

在 `HeaderExchangeChannel` 方法中完成 `DefaultFuture` 创建后, 会将请求通过底层的 Channel 发送出去, 发送过程中会触发 `HeaderExchangeHandler` 的 `sent()` 方法, 在这个方法中会更新 `sent` 字段, 记录发送时间戳, 后续如果出现超时, 会在提示信息中展示该时间戳.

过一段时间后, `Consumer` 端回收到对端的完整响应, 会根据线程模型来决定在什么时机转到业务线程池执行 (也就是 `DefaultFuture` 中的 `executor`);

当响应传递到 `HeaderExchangeHandler` 后, 会调用 `handleResponse` 方法处理, 在这个方法中调用了 `DefaultFuture` 的 `received()` 方法, 这个方法会找到与这次请求相关联的 DefaultFuture 对象, 然后调用其 `doReceived()` 方法, 将 `DefaultFuture` 设置为完成状态.

## 4	超时处理流程

在创建 `DefaultFuture` 时, 会调用 `timeoutCheck()` 方法创建 `TimeoutCheckTask` 定时任务, 并添加到时间轮中:

```java
private static void timeoutCheck(DefaultFuture future) {  
    TimeoutCheckTask task = new TimeoutCheckTask(future.getId());  
	// TIME_OUT_TIMER 是一个用于存放超时检查任务的时间轮, 是 DefaultFuture 的静态属性;
    future.timeoutCheckTask = TIME_OUT_TIMER.get().newTimeout(task, future.getTimeout(), TimeUnit.MILLISECONDS);  
}
```

当响应超时后, 会构造超时异常的 Response 后调用 `DefaultFuture` 的 `received()` 方法.

```java
// org.apache.dubbo.remoting.exchange.support.DefaultFuture.TimeoutCheckTask#notifyTimeout
private void notifyTimeout(DefaultFuture future) {  
    // create exception response.  
    Response timeoutResponse = new Response(future.getId());  
    // set timeout status.  
    timeoutResponse.setStatus(future.isSent() ? Response.SERVER_TIMEOUT : Response.CLIENT_TIMEOUT);  
    timeoutResponse.setErrorMessage(future.getTimeoutMessage(true));  
    // handle response.  
    DefaultFuture.received(future.getChannel(), timeoutResponse, true);  
}
```


---