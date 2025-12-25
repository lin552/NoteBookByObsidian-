---
创建时间: "2025-12-23 14:48:48"
作者: wangxiaoming
tags:
---
#### 一、加载阶段错误

这类错误发生在 `System.loadLibrary()`或 `System.load()`调用时，直接导致应用崩溃（`UnsatisfiedLinkError`或其子类）。
##### 1）`UnsatisfiedLinkError: dalvik.system.PathClassLoader`找不到 SO 文件
1️⃣ 错误信息
```java
Couldn't load xxx from loader dalvik.system.PathClassLoader...: findLibrary returned null
```
2️⃣ 原因
- SO 文件未打包进 APK（如 `lib/arm64-v8a/`目录下缺失对应架构的 SO）。
- APK 中 SO 路径错误（如放在 `jniLibs`但 Gradle 未正确配置，或命名错误）。
- 设备架构与 SO 不匹配（如设备是 `arm64-v8a`，但 APK 只有 `armeabi-v7a`的 SO）。
3️⃣ 排查
- 解压 APK 检查 `lib/<架构>/`目录是否存在目标 SO。
- 用 `adb shell getprop ro.product.cpu.abi`确认设备架构，确保 APK 包含对应架构的 SO（通过 `build.gradle`的 `abiFilters`配置）。

##### 2）`UnsatisfiedLinkError: dlopen failed: couldn't find dependent library`
1️⃣ 错误信息
```java
dlopen failed: library "libxxx.so" not found
```
2️⃣ 原因
SO 依赖其他 SO 库（如 MMKV 依赖系统的 `libc++_shared.so`，或第三方库依赖 `liblog.so`），但依赖的库未找到。
3️⃣ 排查
- 使用 `readelf -d libxxx.so | grep NEEDED`查看 SO 的依赖项（需 Linux/Mac 环境）。
- 确保所有依赖的 SO 都打包进 APK（或系统已自带，如 `libc++_shared.so`需手动打包或通过 Gradle 自动引入）。

##### 3）`UnsatisfiedLinkError: dlopen failed: file too short`
1️⃣ 错误信息
```java
dlopen failed: file too short: "/data/app/.../libxxx.so"
```
2️⃣ 原因
SO 文件损坏或未完整写入（如 APK 安装时解压失败，或文件传输中断）。
3️⃣ 排查
- 对比 APK 中 SO 文件的大小与官方提供的原始 SO 文件（如从 MMKV 官网下载的 SO）。
- 重新打包 APK 或重新下载完整的 SO 文件。

##### 4）`UnsatisfiedLinkError: dlopen failed: has text relocations`
1️⃣ 错误信息
```java
dlopen failed: /data/app/.../libxxx.so: has text relocations
```
2️⃣ 原因
SO 文件中存在「文本重定位」（Text Relocations），这在 Android 6.0（API 23）以上被禁止（出于安全考虑）。
3️⃣ 排查
- 检查 SO 编译选项，确保禁用文本重定位（如 NDK 编译时添加 `-fPIC`和 `--no-undefined`标志）。
- 升级 SO 库到支持 Android M 以上的版本（如使用最新版 `MMKV、FFmpeg` 等）。

##### 5) `UnsatisfiedLinkError: dlopen failed: Permission denied`
1️⃣ 错误信息
```java
dlopen failed: couldn't map ".../libxxx.so" segment 1: Permission denied
```
2️⃣ 原因
- SO 文件权限不足（如系统应用中 SO 文件仅所有者可读写，其他用户无权限）。
- `SELinux` 策略限制（系统应用被禁止访问 `/system/app/`下的 SO 文件）。
- 系统分区挂载为只读（罕见，如 recovery 模式下）。
3️⃣ 排查
- 检查 SO 文件权限（如 `chmod 644 libxxx.so`）。
- 临时关闭 SELinux 验证（`setenforce 0`），确认是否为 SELinux 限制。