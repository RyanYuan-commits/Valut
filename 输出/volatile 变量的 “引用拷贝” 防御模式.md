---
type: permanent
banner: 附件/Banner/pexels-clive-kim-2523249-4220967.jpg
---
当一个变量被声明为 `volatile` 且可能被多个线程异步修改（尤其是置空）时，在方法内部使用时，需要先将其**赋值给局部变量**，再进行后续处理；这种处理方式称为 copy reference；

`volatile` 变量不保证原子性，如果在 `if(xxx != null)` 的检查后，该变量被其他线程设置为 `null`，会造成意料之外的后果。

```java
public class ConnectionManager {
    private volatile Connection connection;

    public void execute(Task task) {
        // Copy reference
        Connection localConn = this.connection; 
	      
        if (localConn != null && localConn.isActive()) {
            try {
                localConn.send(task);
            } catch (Exception e) {
                // 处理异常
            }
        }
    }
}