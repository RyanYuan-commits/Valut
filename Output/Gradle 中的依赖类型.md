---
type: permanent
banner: Assets/Banner/pexels-photo-9160637.jpeg
---
---

**关键词**: Gradle, 依赖管理

---

## 1	外部模块依赖

用于从外部仓库 (如 Maven) 下载库, 是最为常用的方式.

```groovy
implementation 'com.google.guava:guava:31.1-jre'
implementation group: 'com.google.guava', name: 'guava', version: '31.1-jre'
```

## 2	项目依赖

用于引用当前构建中的其他模块, 在多项目构建中很常见, 使用 `project()` 方法, 指向另一个子项目的路径.

```groovy
implementation project(':data')
```

## 3	文件依赖

用于引用本地硬盘上的特定文件, 通常用于那些没有发布到 Maven 仓库的本地库.

```groovy
implementation files('libs/something.jar')
implementation fileTree(dir: 'libs', include: ['*.jar'])
```

---