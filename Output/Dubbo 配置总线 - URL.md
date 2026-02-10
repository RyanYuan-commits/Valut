---
type: permanent
banner: Assets/Banner/pexels-erike-fusiki-58866350-8150291.jpg
---
---

**关键词**: dubbo

---

URL (Uniform Resource Locato, 统一资源定位符), 是互联网的唯一资源定位标志, 也就是指网络地址; 其本质上是特殊格式的字符串, 一个标准的 URL 包含如下几个部分:

```
protocol://username:password@host:port/path?key=value&key=value
```

- `protocol`: URL 的协议. 常见有 HTTP, HTTPS 协议;
- `username/password`: 用户名/密码. HTTP Basic Authentication 中多会使用在 URL 的协议之后直接携带用户名和密码的方式.
- `host/port`: 主机/端口. 在实践中一般会使用**域名**, 而不是使用具体的 host 和 port.
- `path`: 请求的路径.
- `parameters`: 参数键值对. 一般在 GET 请求中会将参数放到 URL 中, POST 请求会将参数放到请求体中.

Dubbo 中**任意的一个实现**都可以抽象为一个 URL, Dubbo 使用 URL 来统一描述了**所有对象和配置信息**, 并贯穿在整个 Dubbo 框架之中.

```
dubbo://172.17.32.91:20880/org.apache.dubbo.demo.DemoService?anyhost=true&application=dubbo-demo-api-provider&dubbo=2.0.2&interface=org.apache.dubbo.demo.DemoService&methods=sayHello,sayHelloAsync&pid=32508&release=&side=provider&timestamp=1593253404714dubbo://172.17.32.91:20880/org.apache.dubbo.demo.DemoService?anyhost=true&application=dubbo-demo-api-provider&dubbo=2.0.2&interface=org.apache.dubbo.demo.DemoService&methods=sayHello,sayHelloAsync&pid=32508&release=&side=provider&timestamp=1593253404714
```

在 Dubbo 中, URL 的构造方法为:

```java
// org.apache.dubbo.common.URL#URL
public URL(String protocol, String username, String password, String host, int port, String path, Map<String, String> parameters) {  
    if (StringUtils.isEmpty(username) && StringUtils.isNotEmpty(password)) {  
        throw new IllegalArgumentException("Invalid url, password without username!");  
    }  
	  
    this.urlAddress = new PathURLAddress(protocol, username, password, path, host, port);  
    this.urlParam = URLParam.parse(parameters);  
    this.attributes = null;  
}
```



---