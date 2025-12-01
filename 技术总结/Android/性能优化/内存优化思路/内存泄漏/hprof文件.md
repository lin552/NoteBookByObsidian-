---
创建时间: "2025-11-28 17:47:53"
作者: wangxiaoming
tags:
---
#### 一、hprof文件简介

`hprof`（Heap Profile）是 **Java虚拟机（JVM）/Android Dalvik/ART虚拟机**​ 生成的内存快照文件，记录了某一时刻 **Java堆内存的详细使用情况**，包括：
- 所有对象实例的类型、数量、内存占用；
- 类信息（类名、父类、字段）；
- GC Roots（垃圾回收根节点，如静态变量、活动线程栈引用）；
- 对象引用关系（谁引用了谁，是否存在循环引用或泄漏）。

在Android开发中，`hprof`文件常用于分析 **内存泄漏**（如Activity/Fragment未释放）、**内存溢出（OOM）**、**大对象占用**​ 等问题。

#### 二、hprof文件生成方式
##### 1) 命令行工具（adb）
**步骤**：
1. 连接设备，通过`adb shell`获取进程ID（PID）：
```bash
adb shell ps | grep 包名  # 如 adb shell ps | grep com.xiaopeng.musicradio
```
2. 生成hprof文件（保存到设备）：
```bash
adb shell am dumpheap <PID> /data/local/tmp/heap.hprof  # 如 adb shell am dumpheap 18100 /data/local/tmp/heap.hprof
```
拉取到本地：
```bash
adb pull /data/local/tmp/heap.hprof ./local_heap.hprof
```
##### 2)代码触发（主动埋点）
通过`Debug.dumpHprofData()`在代码中手动生成（需`WRITE_EXTERNAL_STORAGE`权限）：
```java
try {
    File file = new File(getExternalFilesDir(null), "manual_dump.hprof");
    Debug.dumpHprofData(file.getAbsolutePath()); // 生成hprof文件
} catch (IOException e) {
    e.printStackTrace();
}
```

#### 三、hprof文件查看
##### 1）Android Studio 查看
**特点**：集成于Android Studio，无需额外安装，可直接关联代码，可视化分析内存占用。
**使用步骤**：
1. 打开Android Studio → 点击 **File → Open**，选择生成的`hprof`文件（若提示“Convert to Android format”，点击确认转换，因为Android的hprof格式与标准JVM略有差异）；
2. 进入 **Memory Profiler**​ 界面，主要面板说明：
    - **Histogram（直方图）**：按类名统计对象数量和内存占用（可按`Retained Size`排序，找大对象）；
    - **Dominator Tree（支配树）**：展示对象间的引用关系，识别“内存霸主”（如占用内存最多的对象及其引用链）；
    - **Top Consumers（主要消费者）**：按包名/类分组，快速定位占用内存的模块；
    - **References（引用链）**：选中对象后，查看谁引用了它（右键对象 → **Find Path to GC Roots**​ → **Exclude weak references**，找强引用泄漏源）。
**关键操作**
- 搜索特定类：在Histogram顶部搜索框输入类名（如`MainActivity`）；
- 对比两次快照：生成两个时间点的hprof（如操作前后），通过 **Compare with previous**​ 按钮，找出新增/未释放的对象（定位泄漏）。

##### 2）MAT查看
**步骤1：找到 `hprof-conv`工具**​
`hprof-conv`位于 Android SDK 的 `platform-tools`目录，路径示例：
- **Windows**：`C:\Users\你的用户名\AppData\Local\Android\Sdk\platform-tools\hprof-conv.exe`
- **Mac**：`~/Library/Android/sdk/platform-tools/hprof-conv`
- **Linux**：`~/Android/Sdk/platform-tools/hprof-conv`
**步骤2：转换 `hprof` 文件**​
用命令行将 Android 格式的 `hprof` 转换为标准格式：
```bash
# 语法：hprof-conv [原始Android hprof文件] [转换后标准hprof文件]
hprof-conv hprof_com.xiaopeng.musicradio_7343_2025-11-28-10-31-06.hprof converted_heap.hprof
```
 **步骤3：用转换后的文件分析**​
转换完成后，用 **MAT**​ 或 **Android Studio Profiler**​ 打开 **`converted_heap.hprof`**（标准格式），即可正常分析。