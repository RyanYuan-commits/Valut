---
created: 2025-11-23
type: permanent
banner: "[[pexels-picjumbo-com-55570-4457409.jpg]]"
---
# 🌐 核心观点

BigDecimal 的格式化输出在需要控制小数点位数, 千分位分隔符的场景下非常重要, 如财务报表, 金额显示.

---

# 🔖 详细解释

## 1	格式化实例方法

### 1.1	获取格式化字符串

```java
// 获取原生字符串
BigDecimal smallNum = new BigDecimal("0.000000123456789");
System.out.println("小数的 toPlainString(): " + smallNum.toPlainString()); // 输出: 0.000000123456789

// 获取工程计数法字符串
BigDecimal num2 = new BigDecimal("0.000000123456789");
System.out.println("Engineering: " + num2.toEngineeringString()); // // 输出: 123.456789E-9
```

### 1.2	设置标度值与精度

通过 stripTrailingZeros 方法去除末尾多余的 0:

```java
BigDecimal withZeros = new BigDecimal("123.45000");
System.out.println("去零后: " + withZeros.stripTrailingZeros());
```

通过 setScale 方法设置精度, 入参是配置的标度值和舍入方式:

```java
BigDecimal number = new BigDecimal("123.456789");
BigDecimal rounded = number.setScale(2, java.math.RoundingMode.HALF_UP);
```

## 2	外部 Format 对象

### 2.1	NumberFormat

```java
BigDecimal amount = new BigDecimal("12345.6789");

// 获取当前语言环境的数字格式（如中文环境会有千分位）
NumberFormat formatter = NumberFormat.getNumberInstance();
formatter.setMinimumFractionDigits(2); // 最小小数位数
formatter.setMaximumFractionDigits(2); // 最大小数位数
System.out.println(formatter.format(amount)); // 在中文环境下输出: 12,345.68


// 获取货币格式（会自动添加货币符号）
NumberFormat currencyFormatter = NumberFormat.getCurrencyInstance(Locale.US);
System.out.println(currencyFormatter.format(amount)); // 输出: $12,345.68

NumberFormat chinaCurrencyFormatter = NumberFormat.getCurrencyInstance(Locale.CHINA);
System.out.println(chinaCurrencyFormatter.format(amount)); // 输出: ￥12,345.68
```

### 2.2	DecimalFormat

DecimalFormat 是非线程安全的重量级对象, 内存占用大 + 初始化开销大, 使用的时候要注意缓存 + 线程安全.

```java
import java.math.BigDecimal;
import java.text.DecimalFormat;

BigDecimal amount = new BigDecimal("12345.6789");

// 模式字符串：
// # - 可选数字 (如果为0则不显示)
// 0 - 强制数字 (如果为0则显示0)
// , - 千分位分隔符
// . - 小数点
DecimalFormat df = new DecimalFormat("#,##0.00");
String formatted = df.format(amount);
System.out.println(formatted); // 输出: 12,345.68

// 其他模式示例
DecimalFormat df2 = new DecimalFormat("￥#,##0.00;￥-#,##0.00");
System.out.println(df2.format(amount)); // 输出: ￥12,345.68

BigDecimal negativeAmount = new BigDecimal("-12345.6789");
System.out.println(df2.format(negativeAmount)); // 输出: ￥-12,345.68

// 百分比格式
BigDecimal rate = new BigDecimal("0.8567");
DecimalFormat percentDf = new DecimalFormat("0.00%");
System.out.println(percentDf.format(rate)); // 输出: 85.67%
```

## 3	String.format 方法

简洁, 适合快速格式化.

```java
import java.math.BigDecimal;

BigDecimal amount = new BigDecimal("12345.6789");

// %f - 浮点数
// , - 启用千分位分隔符
// .2 - 小数点后2位
String formatted = String.format("%,.2f", amount);
System.out.println(formatted); // 输出: 12,345.68

// 更复杂的格式
String message = String.format("总金额：￥%,.2f 元", amount);
System.out.println(message); // 输出: 总金额：￥12,345.68 元
```

---

# 📚 参考内容

