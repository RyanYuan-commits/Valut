---
created: 2025-11-22
type: permanent
banner: https://www.notion.so/images/page-cover/nasa_the_blue_marble.jpg
---
# 🌐 核心观点

Netty 将有接收关系的监听通道和传输通道之间的关系描述为父子通道.

---

# 🔖 详细解释

操作系统底层的 Socket 文件描述符分为两类:

- **连接监听类型**: 处于服务器端, 负责接收客户端的套接字连接; 一个 "连接监听类型" 的 Socket 描述符可以接受成千上万的传输类的 socket 文件描述符;
	
- **数据传输类型**: 数据传输类的 socket 描述符负责传输数据, 同一条 TCP 的 Socket 传输链路, 在服务器和客户端, 都分别会有一个与之相对应的数据传输类型的 Socket 文件描述符.

Netty 将有**接收关系**的监听通道和传输通道, 叫做父子通道: 负责服务器连接监听和接收的监听通道称为父通道, 每一个接收到的传输类通道称为子通道, 如 NioServerSocketChannel 和其建立的 NioSocketChannel 之间, 就是父子通道的关系.

---

# 📚 参考内容

