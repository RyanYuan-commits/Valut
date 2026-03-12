---
type: permanent
banner: Assets/Banner/pexels-jeremy-bishop-1260133-2524874.jpg
---
---

**关键词**: Dubbo, Exchange

---

类的全限定名: org.apache.dubbo.remoting.exchange.Request

```java
public class Request {  

	// 用于生成请求的自增 ID, 当递增到 Long.MAX_VALUE 之后, 
	// 会溢出到 Long.MIN_VALUE, 可以继续使用该负数作为消息 ID, 随机生成初始值.
	private static final AtomicLong INVOKE_ID;  

	// 请求的唯一标识, 当有返回时, 通过 mId 来确定返回值属于哪次请求;
	private final long mId;  

	// 请求的协议版本号.
	private String mVersion;  

	// 请求需要一个返回值
	private boolean mTwoWay = true;  
	
	// 请求是事件或者单向请求
	private boolean mEvent = false;  
	
	// 请求发送到 Server 之后, 由 Decoder 将二进制数据解码成 Request 对象;
	// 如果解码环节遇到异常, 则会设置该标识, 然后交由其他 ChannelHandler 根据该标识做进一步处理
	private boolean mBroken = false;  
	
	private int mPayload;  

	// 请求体, 可以是任何 Java 类型的对象,也可以是 null
	private Object mData;

}
```

---