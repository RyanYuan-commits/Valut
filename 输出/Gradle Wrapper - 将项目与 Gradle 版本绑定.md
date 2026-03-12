---
created: 2025-11-23
type: permanent
---
# 🌐 核心观点

介绍 Gradle Wrapper, 用于将 Gradle 版本与项目绑定, 避免因为环境不同造成的编译产物不一致问题.

---

# 🔖 详细解释

Gradle 包装器, 是标准化的在项目中集成和执行 Gradle 构建的方式; 允许新开发者通过 `./gradlew` 命令来完成项目的初始化, Wrapper 会搞定 Gradle 的下载和配置.

## 1	构成部分

一个包含 Gradle Wrapper 的项目通常有以下文件：

- `gradlew`: 在 macOS/Linux 开发环境下使用的 shell 脚本;
	
- `gradlew.bat`: 在 Windows 开发环境下使用的批处理脚本;
	
- `gradle/wrapper/gradle-wrapper.jar`: Wrapper 的核心逻辑, 负责下载 Gradle;
	
- `gradle/wrapper/gradle-wrapper.properties`: 关键配置文件, 定义了要下载的 Gradle 版本和下载地址.

## 2	常用命令

```bash
# 使用 Gradle 为当前项目生成 Wrapper 文件
gradle wrapper

# 修改 Wrapper 的版本, 会更新 gradle-wrapper.properties 文件
./gradlew wrapper --gradle-version [VERSION]
```

---

# 📚 参考内容

