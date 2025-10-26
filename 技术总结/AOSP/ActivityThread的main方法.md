---
创建时间: "2025-08-20 14:53:52"
作者: wangxiaoming
tags:
---
`ActivityThread`的 `main`方法是 ​**应用进程的入口方法**​（`JVM` 启动后执行的第一个用户代码），负责初始化主线程环境、连接系统服务（如 `AMS`）、启动消息循环，并驱动应用的组件运行。以下是其核心步骤的详细拆解（基于 Android 13 源码逻辑）：

#### 1、初始化主线`Looper`（消息循环）
主线程（`UI` 线程）需要一个 `Looper`来处理消息队列（如点击事件、生命周期回调等），`main`方法的第一步即为创建主线程的 `Looper`：
```java
public static void main(String[] args) {
    // 步骤1：准备主线程 Looper（核心！）
    Looper.prepareMainLooper(); 

    // 步骤2：创建 ActivityThread 实例（当前进程的“主线程管理器”）
    ActivityThread thread = new ActivityThread();

    // 步骤3：关联当前线程（主线程）与 ActivityThread 实例
    thread.attach(false, startSeq);

    // 步骤4：启动消息循环（阻塞等待消息）
    Looper.loop(); 

    // 步骤5：退出循环时的清理（理论上不会执行到，进程结束前触发）
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```
#### 2.关键步骤详解
##### (1)`Looper.prepareMainLooper()`
主线程的 `Looper`是全局唯一的，通过 `Looper.prepareMainLooper()`创建。与普通线程的 `Looper.prepare()`不同，它标记当前线程为“主线程”，并创建一个 `MessageQueue`（消息队列）。
- ​**作用**​：为主线程提供消息处理能力（如处理 `Activity`生命周期、`View`事件等）。
- **注意**​：主线程的 `Looper`不能通过 `quit()`或 `quitSafely()`终止（否则会导致 `ANR`或崩溃），只能随进程结束而销毁。

##### (2)创建`ActivityThread`实例
`ActivityThread`是主线程的“管理者”，内部维护了与 `AMS` 的 Binder 连接、已启动的 `Activity/Service`等组件的引用，以及处理系统消息的 `Handler`（内部类 `H`）。
##### (3)thread.attach(...)：绑定`AMS`与进程初始化
`attach`方法是连接当前进程与系统服务（`AMS`）的核心逻辑，参数 `false`表示当前进程是“应用进程”（非系统进程），`startSeq`是 `AMS` 分配的进程启动序列号（用于同步）。

`attach`方法内部通过 ​**Binder 调用**​ 触发 `AMS` 的 `attachApplication`方法，具体完成以下操作：
###### a.初始化进程级资源
- 绑定 `Context`：创建进程的 `ContextImpl`（应用级上下文），后续组件的 `Context`均基于此。
- 连接系统服务：通过 `ActivityManager.getService()`获取 `AMS` 的 Binder 代理对象（`IActivityManager`），用于后续与 `AMS` 通信。
###### b.触发Application的创建与初始化
`AMS` 收到 `attachApplication`调用后，会检查应用的 `AndroidManifest.xml`中声明的 `Application`类，通过反射创建实例，并依次调用其 `attachBaseContext()`和 `onCreate()`方法（详见前序问题中自定义 `Application`的流程）。

##### (4)`Looper.loop()`:启动消息循环
主线程进入 `Looper.loop()`后，会阻塞并持续从 `MessageQueue`中取出消息，通过 `H`（`ActivityThread`内部的 `Handler`）分发处理。

**消息类型示例**​：
- `H.LAUNCH_ACTIVITY`：启动 `Activity`（触发 `Activity`的 `onCreate`等生命周期）。
- `H.BIND_SERVICE`：绑定 `Service`。
- `H.STOP_ACTIVITY`：停止 `Activity`。
- `H.UNBIND_SERVICE`：解绑 `Service`。

#### 3.多进程场景的特殊处理
若应用通过 `android:process`为组件声明了独立进程（如 `:remote`），每个进程会独立启动一个 `JVM`，并执行自己的 `ActivityThread.main()`方法。此时：
- 每个进程有独立的 `Looper`、`ActivityThread`实例和 `Application`对象。
- `AMS` 会为每个进程单独调用 `attachApplication`，触发该进程内 `Application`的初始化。

#### 总结： **main 方法的核心职责**
`ActivityThread.main()`是应用进程的“启动引擎”，核心完成以下任务：

1. 为主线程初始化消息循环（`Looper`），使其具备处理事件的能力。
2. 创建 `ActivityThread`实例，作为主线程的管理者。
3. 通过 Binder 连接 `AMS`，触发进程级资源初始化（如 `Context`、系统服务绑定）和 `Application`的创建。
4. 启动消息循环，持续处理 `AMS` 下发的组件操作指令（如启动 `Activity`、绑定 `Service`等）。
理解 `main`方法的流程，有助于定位应用启动卡顿、组件初始化异常等问题（例如，耗时操作放在 `Application.onCreate()`中可能导致 `ANR`，因为 `AMS` 会在 `onCreate`完成后才允许启动其他组件）。