---
type: permanent
banner: Assets/Banner/Pasted image 20251122105012.png
---
---

**关键词**: Gradle, 依赖管理

---

Gradle 内置了 `dependencies` 任务, 可以从命令行生成依赖关系树, 可以确定构建依赖过程中传递依赖项的解析情况.

## 1	dependencies 的输出注释

```bash
$ ./gradlew :app:dependencies

> Task :app:dependencies

------------------------------------------------------------
Project ':app'
------------------------------------------------------------

annotationProcessor - Annotation processors and their dependencies for source set 'main'.
No dependencies

compileClasspath - Compile classpath for source set 'main'.
\--- com.fasterxml.jackson.core:jackson-databind:2.17.2
     +--- com.fasterxml.jackson.core:jackson-annotations:2.17.2
     |    \--- com.fasterxml.jackson:jackson-bom:2.17.2
     |         +--- com.fasterxml.jackson.core:jackson-annotations:2.17.2 (c)
     |         +--- com.fasterxml.jackson.core:jackson-core:2.17.2 (c)
     |         \--- com.fasterxml.jackson.core:jackson-databind:2.17.2 (c)
     +--- com.fasterxml.jackson.core:jackson-core:2.17.2
     |    \--- com.fasterxml.jackson:jackson-bom:2.17.2 (*)
     \--- com.fasterxml.jackson:jackson-bom:2.17.2 (*)
...
```

- `(*)`: 表示传递依赖子树重复出现, Gradle 将只展示子树的根;
	
- `(c)`: 表示该元素是依赖约束而非真实的依赖项;
	
- `(n)`: 表示无法解析的依赖项或依赖项配置. 

## 2	指定依赖项配置打印

可以通过 `--configuration` 参数来实现仅打印某一个依赖项配置的依赖信息.

```bash
$ gradle -q dependencies --configuration testRuntimeClasspath
```

---