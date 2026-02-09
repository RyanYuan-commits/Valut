---
type: permanent
banner: Assets/Banner/Pasted image 20251122105012.png
---
---

**关键词**: Dubbo, 异步, Exchange

---

```java
// org.apache.dubbo.remoting.exchange.support.DefaultFuture
public class DefaultFuture extends CompletableFuture<Object> {  
  
    private static final ErrorTypeAwareLogger logger = LoggerFactory.getErrorTypeAwareLogger(DefaultFuture.class);  
  
    /**  
     * in-flight channels     
     */    
    private static final Map<Long, Channel> CHANNELS = new ConcurrentHashMap<>();  
	  
    /**  
     * in-flight requests     
     */    
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
	  
    private final long start = System.currentTimeMillis();  
	  
    private volatile long sent;  
	  
    private Timeout timeoutCheckTask;  
	  
    private ExecutorService executor;
	
	// ......
	
}
```

`DefaultFuture` 继承 `CompletableFuture`, 在其中维护了两个静态的 `Map`:

- `CHNNELS`: 负责请求与 `Channel` 之间的关联关系, Key 为请求 ID, Value 为发送请求的 `Channel`;
	
- `FUTURES`: 负责请求与 `DefaultFuture` 之间的关联关系, Key 为请求 ID, Value 为请求对应的 `DefaultFuture`.



---