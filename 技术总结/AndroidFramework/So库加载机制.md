---
创建时间: "2025-06-13 18:06:22"
作者: wangxiaoming
tags:
---
#### 一、基础概念与架构
1. **SO库本质**​
    - 动态链接库（Shared Object），ELF格式，包含代码段（.text）、数据段（.data）及符号表（`.dynsym`）
    - 作用：封装高性能计算模块（如音视频编解码、图像处理），提升代码复用率
2. `ABI`兼容性**​
    - Android支持的CPU架构：`armeabi`、`arm64-v8a`、`x86`等
    - ​**考点**​：如何通过`jniLibs`目录按架构分发SO库？若未提供某架构的SO库会怎样？
#### 二、加载机制与调用规范
1. **加载流程**​
    - Java层：`System.loadLibrary("libname")` → `JNI`层：`dlopen()` → Native层：符号解析与重定位
    - ​**关键点**​：
        - SO库必须放置在`jniLibs/<ABI>/`目录下，否则触发`UnsatisfiedLinkError`
        - 库名强制前缀`lib`和后缀`.so`（如`libnative.so`），不可修改
2. ​**动态加载扩展**​
    - 使用`DexClassLoader`加载外部SO文件，需设置`nativeLibraryPath`
    - ​**考点**​：动态加载与静态加载的适用场景对比？如何避免类加载冲突？
#### 三、`JNI`交互与性能优化
1. **`JNI`调用规范**​
    - C/C++函数命名规则：`Java_packageName_ClassName_methodName`（静态注册）
    - ​**考点**​：若Java方法签名错误，如何通过`javah`或`javac -h`生成正确头文件？
2. ​**性能优化策略**​
    - ​**编译优化**​：`NDK`编译时启用`-O2`优化，使用`LLVM`工具链
    - ​**体积压缩**​：通过`strip`去除调试符号，启用`R8`混淆
    - ​**内存管理**​：避免`JNI`层频繁创建Java对象，使用`NewGlobalRef`缓存`JNI`引用
#### 四、兼容性与调试
1. **多设备适配**​
    - ​**考点**​：如何通过`Build.SUPPORTED_ABIS`动态选择最优SO库？
    - ​**陷阱**​：混合使用不同`NDK`版本编译的SO库可能导致崩溃
2. ​**调试工具链**​
    - ​**Native调试**​：使用`LLDB`或`ndk-stack`解析崩溃日志
    - ​**符号冲突**​：通过`nm`命令检查SO库符号表，解决同名函数冲突
#### 五、安全与工程实践
1. ​**安全加固**​
    - SO库签名校验：防止篡改（如`PackageManager`验证签名）
    - ​**考点**​：如何通过`NativeLoader`实现SO库加密加载？
2. ​**工程规范**​
    - ​**版本管理**​：SO库与`APK`版本严格对齐，避免因版本差异导致功能异常
    - ​**多线程加载**​：确保SO库在主线程外加载，避免阻塞`UI`
#### 六、高频面试题示例
1. ​**原理类**​
    - SO库加载时为何需要`extern "C"`修饰符？（解决C++名称修饰问题）
    - 解释`dlopen`的`RTLD_NOW`与`RTLD_LAZY`模式区别
2. ​**实战类**​
    - 如何实现SO库的热修复？（需结合`DexClassLoader`与`JNI_OnLoad`）
    - 若应用启动时提示`java.lang.UnsatisfiedLinkError`，如何排查？
