---
type: permanent
banner: Assets/Banner/Pasted image 20251122105012.png
---
---

**关键词**: java, 定时任务

---
## 1	使用示例

```java
Timer timer = new Timer();

TimerTask task = new TimerTask() {
	@Override
	public void run() {
		System.out.println("任务执行时间: " + new java.util.Date());
	}
};

timer.schedule(task, 1000);
```

---