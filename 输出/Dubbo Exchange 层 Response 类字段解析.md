---
type: permanent
banner: Assets/Banner/pexels-maxravier-3331094.jpg
---
---

**关键词**: Dubbo, Exchange

---

类的全限定名: org.apache.dubbo.remoting.exchange.Response

```java
public class Response {

	// 响应的唯一标识, 与 Request 的一致 
	private long mId = 0;  
	
	// 协议版本号, 与 Request 一致
	private String mVersion;  
	
    // 响应状态码，有OK、CLIENT_TIMEOUT、SERVER_TIMEOUT等10多个可选值
	private byte mStatus = OK;  
	
	private boolean mEvent = false;  
	  
    // 可读的错误响应消息
	private String mErrorMsg;  
	
	// 响应体
	private Object mResult;

}
```

---