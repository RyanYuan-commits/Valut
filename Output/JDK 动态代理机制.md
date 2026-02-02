---
type: permanent
banner:
---
---

**关键词**: Java, 动态代理

---

JDK 动态代理是基于接口代理, 要求被代理的类必须实现至少一个接口.

## 1	使用方式

首先定义一个类, 这个类需要实现至少一个接口:

```java
class UserServiceImpl implements UserService {  
    @Override  
    public void saveUser(String name) {  
        System.out.println("保存用户：" + name);  
    }  
}
```

继承 `InvocationHandler` 接口编写代理逻辑:

```java
class UserServiceProxy implements InvocationHandler {
    private Object target;
	
    public UserServiceProxy(Object target) {
        this.target = target;
    }
	
    public UserServiceProxy() {}
	
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
        System.out.println("Before method: " + method.getName());  
        Object result = null;  
        if (target != null) result = method.invoke(target, args);  
        System.out.println("After method: " + method.getName());  
        return result;  
    }
}
```

通过 `Proxy.newProxyInstance()` 方法生成代理对象:

```java
public static void main(String[] args) {  
    // 需要目标类  
    UserService target = new UserServiceImpl();  
    UserService proxy1 = (UserService) Proxy.newProxyInstance(  
            target.getClass().getClassLoader(),  
            target.getClass().getInterfaces(),  
            new UserServiceProxy(target)  
    );  
    proxy1.saveUser("test1");  
  
    // 直接代理接口  
    UserService proxy2 = (UserService) Proxy.newProxyInstance(  
            UserService.class.getClassLoader(),  
            new Class[]{UserService.class},  
            new UserServiceProxy()  
    );  
    proxy2.saveUser("test2");  
}
```

## 2	生成的代理类

`Proxy.newProxyInstance()` 生成的代理类继承自 `Proxy` 并实现目标接口:

```java
public final class $Proxy0 extends Proxy implements UserService {
    private InvocationHandler h;
    public $Proxy0(InvocationHandler h) {
        super(h);
    }
    public void saveUser(String username) {
        h.invoke(this, method, new Object[]{username});
    }
}
```

---