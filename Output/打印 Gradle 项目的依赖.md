---
type: permanent
banner: Assets/Banner/pexels-maxravier-3331094.jpg
---
# 🌐 关键词

Gradle, 依赖管理, 依赖分析

---

# 🔖 详细解释

Gradle 提供了工具来浏览依赖管理的结果, 可以生成完整的依赖视图, 确定依赖项的来源, 为何选定了某个版本.

## 1	使用 dependencies 任务列出项目依赖项

Gradle 内置了 `dependencies` 任务, 可以从命令行生成依赖关系树, 可以确定构建依赖过程中传递依赖项的解析情况.

### 1.1	dependencies 的输出注释

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


### 1.2	指定依赖项配置打印

可以通过 `--configuration` 参数来实现仅打印某一个依赖项配置的依赖信息.

```bash
$ gradle -q dependencies --configuration testRuntimeClasspath
```

## 2	使用 dependencyInsight 任务来确定所选版本

### 2.1	任务选项

`dependencyInsight` 任务用于查询单个配置中单个依赖项的信息, 用于分析版本选择的原因和来源. `dependencyInsight` 任务接收如下参数: 

 - `--dependency <dependency>`: 指定要调查的依赖项, 可以提供完整的 `group:name` 或者提供一部分来模糊查询;
	 
 - `--configuration <name>`: 给定一个依赖配置, [Java 插件](https://docs.gradle.org/current/userguide/java_plugin.html#java_plugin)会提供一个默认值 `compileClasspath`; 
	 
 - `--single-path`: 可选参数, 仅将单个路径渲染到依赖项;
	 
 - `--all-variants`: 可选参数, 有关所有变体的信息, 而不仅仅是所选变体的信息.



 ```
 ```

 
---

# 📚 参考内容

