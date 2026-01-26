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

```bash
$ gradle -q dependencyInsight --dependency commons-codec --configuration scm
commons-codec:commons-codec:1.7
  Variant default:
    | Attribute Name    | Provided | Requested |
    |-------------------|----------|-----------|
    | org.gradle.status | release  |           |
   Selection reasons:
      - By conflict resolution: between versions 1.7 and 1.6

commons-codec:commons-codec:1.7
\--- scm

commons-codec:commons-codec:1.6 -> 1.7
\--- org.apache.httpcomponents:httpclient:4.3.6
     \--- org.eclipse.jgit:org.eclipse.jgit:4.9.2.201712150930-r
          \--- scm

A web-based, searchable dependency report is available by adding the --scan option.
```

### 2.2	理解选择原因

对于被选择的依赖, 会有这些标识描述它为什么在:

- `(Absent)`: 表示默认存在, 是最基础的情况. 没有任何特殊情况, 仅仅是因为你在代码中直接引用了它, 或者是别的库引用了它(传递依赖), 所以它自然出现在了依赖树中;
	
- `Was requested : <text>`, 表示按需请求. 明确指定了某个版本, 或者使用了动态版本(如 1. +), Gradle 按照你的要求找到了这个版本. `<text>` 通常是你写的版本号或理由;
	
- `By conflict resolution : between versions <version>`:  表示冲突解决, 不同的库分别依赖了同一个库的不同版本,  Gradle 的默认策略是选最新, 所以这里显示了它通过冲突解决机制, 选中了其中的胜出版本;
	
- `By constraint`: 表示约束限制, 在 Gradle 中使用了 constraints 代码块强行指定了版本. 虽然别的地方可能请求了别的版本, 但约束拥有更高优先级, 所以选了这里指定的版本;
	
- `By ancestor`:  表示祖先强制, 某个上游依赖使用了 strictly(严格版本)关键字;
	
- `Forced`: 表示强制执行, 是最高级别的强制. 通常是因为你在 `resolutionStrategy` 中配置了 `force 'xxx:version'`, 或者使用了强制平台, Gradle 必须无条件使用这个版本.

如果一个依赖被拒绝, Gradle 也会显示原因:

- `Was requested : didn’t match versions <versions>`: 表示动态版本不匹配, 在请求的是动态版本( 如 `1.+` ), Gradle 在仓库里找到了这个版本, 但它不在 `1.+` 的范围内, 或者不符合你的要求, 所以被扔掉了.
- 


---

# 📚 参考内容

