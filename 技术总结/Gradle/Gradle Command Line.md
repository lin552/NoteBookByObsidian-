---
创建时间: "2026-03-19 10:06:11"
作者: wangxiaoming
tags:
---
#### 一、构建相关命令
```bash
# 构建所有变体 APK（包括 debug 和 release）
./gradlew assemble

# 构建 debug 版本​ APK
./gradlew assembleDebug

# 构建 release 版本 APK
./gradlew assembleRelease

# 相当于 `clean + assemble + check`，会跑单元测试和 lint
./gradlew build

# 删除 build目录，清理构建缓存
./gradlew clean

# 有些 IDE 支持，相当于 `clean + build`
./gradlew rebuild
```

#### 二、安装与运行相关命令
```bash
# 构建并安装 debug 版本​ 到连接的设备/模拟器
./gradlew installDebug

# 构建并安装 release 版本（需签名）
./gradlew installRelease

# 卸载 debug 版本应用
./gradlew uninstallDebug

# 卸载所有该项目的应用
./gradlew uninstallDebug

```

#### 三、依赖与项目信息命令
```bash
# 查看项目依赖树（包括传递依赖）
./gradlew dependencies

# 查看 `app`模块依赖（可替换成其他模块名）
./gradlew app:dependencies

# 显示 Gradle 项目属性
./gradlew properties

# 列出所有可用任务（包括自定义任务）
./gradlew tasks

# 查看某个任务的帮助信息，比如 `./gradlew help --task assembleDebug`
./gradlew help --task <taskName>
```

#### 四、多渠道/多环境构建（Flavor相关）
如果你的 `build.gradle`配置了 `productFlavors`，可以用：
```bash
# 构建 `paid`flavor 的 debug 包
./gradlew assemblePaidDebug

# 构建 `free`flavor 的 release 包
./gradlew assembleFreeRelease

# 构建 paidflavor 的所有变体（debug + release）
./gradlew assemblePaid
```

#### 五、其他实用命令
```bash
# 查看应用的签名信息（包括 debug 和 release 签名）
./gradlew signingReport

# 构建 Android App Bundle（AAB），用于 Google Play 发布
./gradlew bundle

# 构建 release 版本的 AAB
./gradlew bundleRelease

# 提取混淆规则文件（如果启用了 ProGuard/R8）
./gradlew :app:extractProguardFiles
```