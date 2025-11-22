---
source:
date:
type: input
created: 2025-11-21
---
# 阅读笔记

## 1	基本估算

对象大小 = 对象头 + 实例数据 + 对齐填充

- 对象头, 非数组按照 12 字节算, 数组按照 16 字节算
	
- 实例数据: byte/ boolean: 1 字节, short / char: 2 字节, int / float: 4 字节, long / double: 8 字节, 引用类型 4 字节
	
- 对齐填充: 估算 4 字节

## 2	案例

### 2.1	估算 String 的大小

String 本身: 12 + 4 + 4 = 24

value 数组: 12 + 4 = 16

数组中引用的 byte, 普通字符每个占 1 字节, 其他占 2 字节