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
  
	private static final AtomicLong INVOKE_ID;  
	
	private final long mId;  
	
	private String mVersion;  
	
	private boolean mTwoWay = true;  
	
	private boolean mEvent = false;  
	
	private boolean mBroken = false;  
	
	private int mPayload;  
	
	private Object mData;

}
```

---