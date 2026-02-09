---
type: permanent
banner: Assets/Banner/pexels-faikackmerd-1025469.jpg
aliases:
  - AbstractEndpoint
---
---

**关键词**: Dubbo, Transport

---

```java
public abstract class AbstractEndpoint extends AbstractPeer implements Resetable {  
  
    protected final ErrorTypeAwareLogger logger = LoggerFactory.getErrorTypeAwareLogger(getClass());  
  
    private Codec2 codec;  
  
    private int connectTimeout;
	
}
```

### 2.1	继承关系

`AbstractEndpoint` 继承了 `AbstractPeer`, 并且实现了 `Resetable` 接口, 有重置配置的能力. `AbstractClient` 和 `AbstractServer` 都是 `AbstractEndpoint` 的子类.

```java
public abstract class AbstractEndpoint extends AbstractPeer implements Resetable {}
```

![[AbstractEndpoint 继承关系.png|700]]


### 2.2	构造函数与 Resetable 特性

```java
public abstract class AbstractEndpoint extends AbstractPeer implements Resetable {  
  
    protected final ErrorTypeAwareLogger logger = LoggerFactory.getErrorTypeAwareLogger(getClass());  
  
    private Codec2 codec;  
  
    private int connectTimeout;  
  
    public AbstractEndpoint(URL url, ChannelHandler handler) {  
        super(url, handler);  
        this.codec = getChannelCodec(url);  
        this.connectTimeout =  
                url.getPositiveParameter(Constants.CONNECT_TIMEOUT_KEY, Constants.DEFAULT_CONNECT_TIMEOUT);  
    }
}
```

`AbstractEndpoint` 中维护了一个 `Codec2` 对象和超时时间, 会通过传入的 URL 进行解析和构建.

```java
@Override  
public void reset(URL url) {  
    if (isClosed()) {  
        throw new IllegalStateException(  
                "Failed to reset parameters " + url + ", cause: Channel closed. channel: " + getLocalAddress());  
    }  
  
    try {  
        if (url.hasParameter(Constants.CONNECT_TIMEOUT_KEY)) {  
            int t = url.getParameter(Constants.CONNECT_TIMEOUT_KEY, 0);  
            if (t > 0) {  
                this.connectTimeout = t;  
            }  
        }  
    } catch (Throwable t) {  
        logger.error(INTERNAL_ERROR, "", "", t.getMessage(), t);  
    }  
  
    try {  
        if (url.hasParameter(Constants.CODEC_KEY)) {  
            this.codec = getChannelCodec(url);  
        }  
    } catch (Throwable t) {  
        logger.error(INTERNAL_ERROR, "unknown error in remoting module", "", t.getMessage(), t);  
    }  
}
```

AbstractEndpoint 还实现了 Resetable 接口, 可以根据传入的 URL 重置 AbstractEndpoint 的字段.

---