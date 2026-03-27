---
type: permanent
banner: 附件/Banner/Pasted image 20251122105012.png
---
---

判断狭义上的基本数据类型，也就是 `boolean`、`byte`、`char`、`short`、`int`、`long`、`float`、`double`，可以通过 `java.lang.Class#isPrimitive` Native 方法；

也可以定义广义上的基本数据类型，如 Dubbo 中判断广义上的基本数据类型的方法为：

```java
public static boolean isPrimitives(Class<?> cls) {  
    if (cls.isArray()) {  
        return isPrimitive(cls.getComponentType());  
    }  
    return isPrimitive(cls);  
}  
  
public static boolean isPrimitive(Class<?> cls) {  
    return cls.isPrimitive() || cls == String.class || cls == Boolean.class || cls == Character.class  
            || Number.class.isAssignableFrom(cls) || Date.class.isAssignableFrom(cls);
```

---