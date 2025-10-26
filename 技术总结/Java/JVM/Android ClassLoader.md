---
创建时间: "2025-07-28 20:51:56"
作者: wangxiaoming
tags:
---
Android 类加载机制是 Android 运行时环境（`ART/Dalvik`）的核心组成部分，负责将类文件（`dex` 文件）加载到内存中并生成对应的 `Class` 对象。其核心设计遵循 Java 虚拟机的双亲委派模型，但针对移动端特性进行了优化。以下是关键要点：

#### 一、Android类加载器类型

Android 提供了三种核心类加载器，形成层级结构：
1. ​`BootClassLoader`
    - ​**职责**​：加载 Android 系统核心类库（如 `java.lang`、`android` 包等），由 C/C++ 实现，在系统启动时预加载
    - ​**特点**​：无父加载器，直接由虚拟机调用，应用程序无法直接实例化。
2. ​`PathClassLoader`
    - ​**职责**​：加载已安装 `APK` 中的类（默认路径为 `/data/app`），是应用程序默认的类加载器
    - ​**特点**​：继承自 `BaseDexClassLoader`，`optimizedDirectory` 参数为 `null`，默认使用系统优化目录（`/data/dalvik-cache`）。
3. ​`DexClassLoader`
    - ​**职责**​：动态加载外部 `dex` 文件（如插件化场景），支持从任意路径加载类
    - ​**特点**​：需指定 `optimizedDirectory` 参数，用于存放优化后的 `dex` 文件（`odex`）。

#### 二、双亲委派机制

Android 类加载遵循 ​**双亲委派模型**，其核心流程为：
1. ​**委派顺序**​：子类加载器优先将请求委派给父加载器，父加载器无法加载时子类才尝试加载
    - 例如：`DexClassLoader` 加载类时，先询问 `PathClassLoader`，再委派至 `BootClassLoader`。
2. ​**作用**​：
    - ​**唯一性**​：避免同一类被多个加载器重复加载。
    - ​**安全性**​：防止恶意代码覆盖系统核心类（如自定义 `java.lang.String`）。
    - ​**顺序性**​：确保依赖类优先加载

#### 三、类加载过程

Android 类加载分为三个阶段：
1. ​**加载（Loading）​**​
    - 从 `dex` 文件中读取类数据，生成 `Class` 对象。
    - 支持 `MultiDex` 优化（主 `dex` 加载核心类，附加 `dex` 动态加载）
2. ​**链接（Linking）​**​
    - ​**验证**​：检查 `dex` 文件格式和签名。
    - ​**准备**​：为静态变量分配内存并设置默认值。
    - ​**解析**​：将符号引用转换为直接内存地址。
3. ​**初始化（Initialization）​**​
    - 执行静态代码块（`<clinit>`），完成静态变量赋值。

#### 四、与Java类加载机制的对比
|​**特性**​|​**Java**​|​**Android**​|
|---|---|---|
|​**核心文件格式**​|`.class` 文件|`.dex` 文件（ART 优化为 `.oat`）|
|​**类加载器类型**​|Bootstrap/Extension/Application|BootClassLoader/PathClassLoader/DexClassLoader|
|​**运行时环境**​|JVM（支持 JIT 编译）|ART（AOT 编译）|
|​**安全性**​|依赖安全管理器|类加载器隔离 + 应用签名验证|
#### 五、高级特性与优化
1. ​`MultiDex` 处理​
    - 解决方法数超过 65535 的问题，主 `dex` 加载启动必需类，附加 `dex` 动态加载
2. ​**热修复**​
    - 通过 `DexClassLoader` 加载补丁 `dex`，替换 `PathList` 中的 `dex` 元素
3. ​`InMemoryDexClassLoader`​
    - Android 8.0+ 支持直接从内存加载 `dex`，提升加载效率
   