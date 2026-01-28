---
type: permanent
banner: Assets/Banner/pexels-picjumbo-com-55570-4457409.jpg
---
# 🌐 关键词

Gradle, 构建工具, 依赖管理

---

# 🔖 详细解释

## 1	Gradle 中的生产者与消费者

![[Gradle 中的生产者与消费者.png|900]]

当构建一个库的时候, 我们扮演的是生产者的角色, 当依赖一个库的时候, 你扮演的是一个消费者的角色. Consumer 可以被宽泛的定义为: 依赖其他项目的项目以及声明对特定 artifacts 的依赖的配置.

## 2	在 Gradle 中添加依赖

在 `build.gradle` 中使用 `dependencies {}` 来配置依赖项, 可以添加外部库, 本地 JAR 文件以及多项目构建中的其他项目, 外部依赖通过依赖配置 + 依赖坐标 (包括 GroupId, ArtifactID, Version) 的方式来声明, 类似:

```groovy
dependencies {
    // Configuration Name + Dependency Notation - GroupID : ArtifactID (Name) : Version
    implementation('com.ryan.test:gradle-test:1.0')
}
```

## 3	Gradle 中的依赖类型

### 3.1	外部模块依赖

用于从外部仓库 (如 Maven) 下载库, 是最为常用的方式.

```groovy
implementation 'com.google.guava:guava:31.1-jre'
implementation group: 'com.google.guava', name: 'guava', version: '31.1-jre'
```

### 3.2	项目依赖

用于引用当前构建中的其他模块, 在多项目构建中很常见, 使用 `project()` 方法, 指向另一个子项目的路径.

```groovy
implementation project(':data')
```

### 3.3	文件依赖

用于引用本地硬盘上的特定文件, 通常用于那些没有发布到 Maven 仓库的本地库.

```groovy
implementation files('libs/something.jar')
implementation fileTree(dir: 'libs', include: ['*.jar'])
```

---

# 📚 参考内容

