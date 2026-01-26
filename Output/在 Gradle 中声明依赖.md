---
type: permanent
banner: Assets/Banner/pexels-picjumbo-com-55570-4457409.jpg
---
# 🌐 关键词

Gradle, 构建工具, 依赖管理

---

# 🔖 详细解释

## 1	Gradle 中的生产者与消费者

![[Gradle 中的生产者与消费者.png]]

当构建一个库的时候, 我们扮演的是生产者的角色, 当依赖一个库的时候, 你扮演的是一个消费者的角色. Consumer 可以被宽泛的定义为: 依赖其他项目的项目以及声明对特定 artifacts 的依赖的配置.

## 2	在 Gradle 中添加依赖

在 `build.gradle` 中使用 `dependencies {}` 来添加依赖项, 可以添加外部库, 本地 JAR 文件以及多项目构建中的其他项目.

---

# 📚 参考内容

