---
type: permanent
banner: 附件/Banner/pexels-vadimsadovski-33522604-7049988.jpg
---
---

**关键词**: Raft、选举

---

## 1	心跳 & 提交同步请求

leader 需要周期性的向 follower 节点发送心跳请求，证明自己还健在，同时同步日志提交的进度，心跳请求是单向无阻塞的，leader 无需等待回复。

```json
{
	"term": 5,
	"leaderID": 1,
	"leaderCommit": 10
}
```

接收到心跳请求的一方如果发现请求的任期比自己当前的任期要小，直接无视，反正，则转化为 follower 节点，重置 leader 心跳检测计时器，并且查看 leaderCommit，查看是否有日志可以被提交。

## 2	竞选拉票流程

当 follower 在一段时间内没有收到 leader 节点的同步请求后，会转变为 candidate 向其他 follower 节点广播拉票请求，以上位为 leader。





---