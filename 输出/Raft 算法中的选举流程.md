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

接收到心跳请求的一方如果发现

---