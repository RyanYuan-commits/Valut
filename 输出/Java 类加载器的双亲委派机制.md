---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
---
---

**关键词**: Java, 类加载器, 双亲委派机制 

---

![[双亲委派机制示意图.png|500]]

在 Java 中, 除 Bootstrap 启动类加载器以外, 每个类加载器都有一个父类加载器, 每当一个类加载器收到加载请求后, 会将类加载请求先委派给父加载器处理; 只有当父加载器无法加载时, 才由子加载器尝试加载. 这样就避免了恶意代码替换 JDK 中的核心类库, 确保核心类库的**完整性和安全性**, 同时也避免了类被**重复加载**.

```java
// java.lang.ClassLoader#loadClass(java.lang.String, boolean)
protected Class<?>  loadClass(String name, boolean resolve) throws ClassNotFoundException {
	synchronized (getClassLoadingLock(name)) {
		// 检查类是否已经被当前加载器加载过
		Class<?> c = findLoadedClass(name);
		if (c == null) {
			long t0 = System.nanoTime();
			try {
				if (parent != null) {
					// 检查父类加载器是否加载了这个类
					c = parent.loadClass(name, false);
				} else {
					// 如果没有父类加载器, 检查 BootStrap 加载器是否加载了这个类
					c = findBootstrapClassOrNull(name);
				}
			} catch (ClassNotFoundException e) {
				// ......
			}
			
			if (c == null) {
				// 当前类加载器的加载逻辑
			}
		}
		// 检查是否需要触发解析阶段
		if (resolve) {
			resolveClass(c);
		}
		return c;
	}
}
```

体现在源码上, 当类加载器加载类时, 会不断向上调用父类加载器的 `loadClass()` 方法, 当父类加载器返回 `null` 时候, 才会尝试自己加载.

---