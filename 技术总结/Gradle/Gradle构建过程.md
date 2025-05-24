---
创建时间: 2025-05-21 22:14:38
作者: wangxiaoming
tags:
  - Gradle
---
#### 一、构建生命周期
`Android Gradle` 构建分为三个核心阶段，由 `Gradle` 守护进程（`Gradle Daemon`）驱动：

1. ​**初始化阶段（Initialization）​**​
    - 解析 `settings.gradle` 文件，确定项目结构（多模块项目需在此声明子模块）。
    - 创建 `Gradle` 全局对象和 `Settings` 对象，建立项目层次关系。
2. ​**配置阶段（Configuration）​**​
    - 解析所有模块的 `build.gradle` 文件，生成任务依赖图（DAG）。
    - 应用插件（如 `Android Gradle Plugin`），配置编译选项、依赖项、资源处理规则等。
    - ​**关键类**​：`Project` 对象管理模块配置，`TaskContainer` 管理任务集合。
3. ​**执行阶段（Execution）​**​
    - 按依赖顺序执行任务链（如编译、打包、签名）。
    - 典型任务链：`clean → compileJava → processResources → assembleDebug`。

#### 二、构建任务流程
Android 构建流程可拆解为以下核心步骤：
1. ​**清理（Clean）​**​
    - 删除 `build/` 目录，确保无旧产物残留。
2. ​**编译（Compile）​**​
    - ​**Java/Kotlin 代码**​：通过 `javac` 或 Kotlin 编译器生成 `.class` 文件。
    - ​**资源处理**​：解析 `res/` 目录，生成 `R.java` 和 `resources.arsc`。
    - ​`Dex` 转换：将 `.class` 文件转换为 `Dalvik` 可执行格式（`DEX`），优化 Android 运行时性能。
3. ​**打包（Package）​**​
    - 合并 `DEX`、资源、`AndroidManifest.xml` 生成未签名的 `APK`（`.apk`）。
    - ​**签名（Sign）​**​：使用调试或发布密钥库对 `APK` 进行签名，确保安全性。
    - ​**对齐优化（`Zipalign`）​​：压缩 `APK` 内存占用，提升运行效率。
4. ​**安装与运行（Install/Run）​**​
    - 通过 `adb` 将 `APK` 推送至设备，或直接在 IDE 中运行调试。

#### 三、依赖管理机制
`Gradle` 依赖解析是构建的核心能力之一：
1. **依赖声明**​
    - 在 `build.gradle` 中通过 `dependencies` 块声明依赖（如 `implementation 'com.google.code.gson:gson:2.8.9'`）。
    - 支持本地 JAR、Maven 仓库（`JCenter、Maven Central`）、私有仓库等。
2. ​**依赖解析策略**​
    - ​**传递性依赖**​：自动解析依赖的依赖（需避免版本冲突）。
    - ​**排除规则**​：通过 `exclude group: 'com.example'` 排除冲突模块。
    - ​**版本统一**​：在根 `build.gradle` 中通过 `dependencyResolutionManagement` 强制统一依赖版本。
3. ​**依赖缓存**​
    - 本地缓存路径：`~/.gradle/caches/`，加速重复构建。

#### 四、性能优化实践
1. ​**启用 `Gradle Daemon`*
    - 在 `gradle.properties` 中设置 `org.gradle.daemon=true`，减少 `JVM` 启动开销。
2. ​**并行构建**​
    - 配置 `org.gradle.parallel=true`，允许同时执行独立任务（如多模块编译）。
3. ​**增量编译**​
    - 利用 `annotationProcessor` 和 `kapt` 的增量注解处理，避免全量重新编译。
4. ​**构建缓存**​
    - 启用远程缓存：`org.gradle.caching=true`，复用跨机器构建产物。
5. ​**任务优化**​
    - 使用 `./gradlew --profile` 生成构建耗时报告，针对性优化耗时任务。

#### 五、插件与自定义扩展
1. `Android Gradle Plugin (AGP)`​​
    - 提供 Android 专属任务（如 `assembleRelease`、`lint`），集成 `ProGuard/R8` 混淆、资源压缩等功能。
2. ​**自定义 Task**​
    - 在 `build.gradle` 中扩展任务：
```groovy
task customTask {
    doLast {
        println "Custom build logic"
    }
}
```
1. ​`BuildSrc` 管理**​
    - 在 `buildSrc` 模块中编写 Kotlin/Java 代码，实现可复用的构建逻辑和依赖版本管理

#### 六、常见问题与解决
1. ​**构建速度慢**​
    - 原因：未启用 Daemon、依赖解析耗时、未使用增量编译。
    - 解决：启用 Daemon、分析依赖树（`./gradlew dependencies`）、优化 `AGP` 版本。
2. ​**依赖冲突**​
    - 现象：编译时报 `Duplicate class` 错误。
    - 解决：使用 `./gradlew dependencyInsight --dependency <库名>` 定位冲突，通过 `exclude` 或强制版本解决。
3. ​`APK` 签名失败**​
    - 原因：密钥库路径或密码错误。
    - 解决：检查 `build.gradle` 中的 `signingConfig` 配置。