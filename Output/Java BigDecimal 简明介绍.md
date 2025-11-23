---
created: 2025-11-23
type: permanent
banner: "[[pexels-eliannedipp-4666748.jpg]]"
---
# 🌐 核心观点

Java BigDecimal 类用于解决浮点数计算中的 "精度丢失" 问题, 并提供了方便清晰的格式化方法.

---

# 🔖 详细解释

## 1	背景: 浮点数的精度丢失问题

BigDecimal 的诞生是为了解决浮点数的精度丢失问题, 如:

```java
System.out.println(0.1 + 0.2); // 输出：0.30000000000000004
System.out.println(1.0 - 0.9); // 输出：0.09999999999999998
```

这种误差是由于浮点数在计算机中是使用二进制浮点数形式表示的, 无法精确的表示所有十进制小数; 

为了解决这个问题, BigDecimal 使用非标度值 + 标度值的方式来精确的表示十进制小数, 例如, 123.45 的非标度值为 12345, 标度值为 2, 表示小数点后有两位.

```java
BigDecimal d1 = new BigDecimal("123.45");
BigDecimal d2 = new BigDecimal("123.4500");
BigDecimal d3 = new BigDecimal("1234500");
System.out.println(d1.scale()); // 2,两位小数
System.out.println(d2.scale()); // 4
System.out.println(d3.scale()); // 0
```

## 2	使用 BigDecimal

### 2.1	BigDecimal 的创建

```java
// 使用字符串构造, 所见即所得, 推荐
BigDecimal a = new BigDecimal("0.1");

// 使用 valueOf 方法, 底层对常见数字做了优化
BigDecimal a = BigDecimal.valueOf(0.1);

// 避免使用 double 构造函数, 因为 double 本身的不精确性会被传递进去
BigDecimal a = new BigDecimal(0.1);
System.out.println(a); // 输出：0.1000000000000000055511151231257827021181583404541015625
```

### 2.2	核心算数运算

BigDecimal 通过成员方法支持各种运算逻辑; 由于 BigDecimal 的不可变性, 每次运算均会返回一个新的 BigDecimal 实例.

常见的运算方法有: add, substract, multiply, divide, remainder.

### 2.3	除法运算与舍入模式

当进行除法运算时, 必须指定舍入模式, 否则会抛出 ArithmeticException 异常.

舍入模式定义在 RoundingMode 枚举类中, 常用的舍入模式有: 

- RoundingMode.HALF_UP 四舍五入;
	
- RoundingMode.HALF_EVEN 银行家舍入法;
	
- RoundingMode.UP 向远离 0 的方向舍入;

- RoundingMode.DOWN: 向靠近 0 的方向舍入.

### 2.4	精度控制和比较

可以通过指定标度 + 舍入方式的方法来实现精度控制.

```java
BigDecimal num = new BigDecimal("123.456789");

// 设置小数点后保留 2 位, 四舍五入
BigDecimal rounded = num.setScale(2, RoundingMode.HALF_UP); // 123.46
```

BigDecimal 的 equals 只有当精度值和标度值均相同时才会返回 true, 所以日常使用中一般使用 compareTo 方法来比较两个 BigDecimal, 其返回值为 -1, 0, 1.


---

# 📚 参考内容

