---
created: 2025-11-23
type: permanent
banner: "[[pexels-vadimsadovski-33522604-7049988.jpg]]"
---
# 🌐 核心观点

对 Java 原生序列化的基础描述, 重点关注一下 @Serializable 接口.

---
# 🔖 详细解释

## 1	什么是序列化?

序列化是将 Java 内存中的对象状态转换为字节序列的过程. 反序列化是从字节序列中重新构建 Java 对象的过程. 其核心目的是让对象可以 "跨越" 时间和空间的界限.

- **跨越时间**: 持久化, 将对象存入硬盘, JVM 关闭后再重启, 仍能恢复对象状态.
    
- **跨越空间**: 传输, 将对象通过网络从一个 JVM 发送到另一个 JVM (例如 RMI).

## 2	如何实现: 基础

Java 要求被序列化的类必须实现 `java.io.Serializable` 标记接口.

```java
import java.io.Serializable;

public class User implements Serializable { 

	// 姓名 (会被序列化) 
	public String name; 
	
	// 年龄 (会被序列化) 
	public int age;
	
	public User(String name, int age) {
	    this.name = name;
	    this.age = age;
	    System.out.println("User constructor called!");
	}
	
	@Override
	public String toString() {
	    return "User{" + "name='" + name + '\'' + ", age=" + age + '}';
	}
}

// ====== Use is ======

User user = new User("Alice", 30); String filename = "user.dat";

// 1. 序列化 (写入文件)
try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(filename))) {
	oos.writeObject(user);
	System.out.println("Object serialized: " + user);
} catch (IOException e) {
	e.printStackTrace();
}

// 2. 反序列化 (从文件读取)
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(filename))) {
	User restoredUser = (User) ois.readObject();
	System.out.println("Object deserialized: " + restoredUser);
} catch (IOException | ClassNotFoundException e) {
	e.printStackTrace();
}
```

## 3	运作机制

### 3.1	序列化

- 创建一个 ObjectOutputStream (例如, 包装一个 FileOutputStream).

- 调用 oos.writeObject(myObject).

- JVM 检查 myObject 是否 implements Serializable.

- JVM 写入该对象的 "类信息" (包括类名, serialVersionUID).

- JVM 遍历该对象的所有 非 transient 和 非 static 的成员变量, 将它们的值写入流中.

- 如果某个成员变量也是一个对象, JVM 会递归地对这些对象执行相同的序列化过程.

### 3.2	反序列化

- 创建一个 ObjectInputStream (例如, 包装一个 FileInputStream).

- 调用 Object obj = ois.readObject().

- JVM 从流中读取"类信息".

- JVM 在本地查找这个类 (例如 com.example.User.class).

- 关键: JVM 检查流中的 serialVersionUID 是否与 .class 文件中的 serialVersionUID 完全匹配.

- 如果匹配: JVM 会绕过构造函数 (constructor), 直接在堆上分配内存, 然后用流中的数据填充所有非 transient 的字段, "瞬时"构建出对象.

- 如果不匹配: 抛出 InvalidClassException 异常.

## 4	transient 关键字: 排除字段

使用 `transient` 关键字标记的成员变量, 将不会参与序列化过程.

- 它修饰的字段在反序列化后, 会被赋予该类型的**默认值** (例如 null, 0, false).
    
- 适用场景: 密码, 临时缓存, 数据库连接等不需要保存或不应保存的敏感信息.

```java
import java.io.Serializable;

/**
 * 序列化前: User{username='Bob', password='123456'}
 * 反序列化后: User{username='Bob', password='null'} (password 丢失, 变回默认值 null)
 */
public class UserWithTransient implements Serializable { 

	public String username; 
	
	// 密码是敏感信息, 不应该被序列化 
	public transient String password; 
	
	// 临时值, 恢复为默认值 0
	public transient int tempCacheValue; 
	
	public UserWithTransient(String username, String password) {
	    this.username = username;
	    this.password = password;
	}
	
	@Override
	public String toString() {
	    return "User{" + "username='" + username + '\'' + ", password='" + password + '\'' + '}';
	}
}
```

## 5	serialVersionUID: 版本指纹

### 5.1	什么是 SerialVersionUID?

SerialVersionUID 是一个 private static final long 类型的字段, 是当前类的一个版本标识, 用于确保 "序列化时的类" 和 "反序列化时的类" 是兼容的.

```java
private static final long serialVersionUID = 1L;
```

- 序列化时: JVM 把这个 `serialVersionUID` 连同对象数据一起写入字节流.
	
- 反序列化时: JVM 比较字节流中的 `serialVersionUID` 和本地 `.class` 文件中的 `serialVersionUID`.
	
	- 一致: 反序列化成功.
	
	- 不一致: 抛出 `InvalidClassException` 异常, 拒绝反序列化.

### 5.2	最佳实践

**必须显式声明**: 只要你的类实现了 Serializable, 就一定要手动声明 SerialVersionUID.

如果你不声明, Java 编译器会根据你的类结构 (字段名, 方法签名等) 自动计算一个 SerialVersionUID. 这非常危险! 因为你只要稍微修改一下类 (比如加个 private 方法), 编译器计算出的 ID 就可能变化, 导致无法读取旧数据.

- 首次创建类时, 设为 `1L`.
	
- 如果你做了不兼容的修改 (比如删掉一个字段, 或修改字段类型), 你必须更改 SerialVersionUID, 这样系统会"优雅地"拒绝加载旧数据 (抛出异常), 而不是尝试加载并出错.
	
- 如果你只是做了兼容的修改 (比如新增一个字段), 你可以保持 SerialVersionUID 不变. 反序列化时, 新增的字段会获得默认值.

---

# 📚 参考内容

