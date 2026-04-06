---
type: permanent
banner: 附件/Banner/pexels-jeremy-bishop-1260133-2524874.jpg
banner-x: 32
banner-y: 55
---
`java.net.InetSocketAddress` 是 **Java 网络编程**中基础且核心的一个类，它代表了一个网络端点，即 ip + port；

## 1	核心特点

`InetSocketAddress` 主要职责是为 socket 提供一个具体的连接点或绑定点；

- ==不可变性==：`InetSocketAddress` 一旦创建，其 ip 和 port 就无法被修改，在多线程环境下使用非常安全；
- ==支持未解析状态==：可以基于 hostname 创建，而不立刻查询 dns 获取 ip，这种未解析的地址在通过代理服务器连接时非常有用，可以让代理服务器解析域名，而不是在本地解析。

## 2	构造方法

`InetSocketAddress(int port)`: 创建一个绑定在本地所有可用 IP 地址上的地址，常用于服务端；

`InetSocketAddress(String hostname, int port)`: 根据主机名和端口号创建，调用此方法时，Java 会尝试在后台进行 DNS 解析以获取 IP 地址，如果不希望触发自动解析的话，可以通过 `createUnresolved(String hostname, int port)` 创建；

`InetSocketAddress(InetAddress addr, int port)`: 直接通过已有的 InetAddress 对象和端口号创建。

## 3	核心组件：InetAddress

`InetSocketAddress` 实质上是对 `InetAddress` 和一个 `int` 类型端口号的封装；

`InetAddress` 代表互联网协议中的 ip 地址，不含有端口信息，其有两个子类 `Inet4Address` 和 `Inet6Address` 分别表示 ipv4 和 ipv6 地址。