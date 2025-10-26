---
创建时间: 2025-07-26 08:08:56
作者: wangxiaoming
tags:
  - AOSP
  - AMS
---
Android 的 ​`AMS`（`ActivityManagerService`）​**​ 是系统核心服务之一，负责管理应用生命周期、进程调度、任务栈等关键功能。以下是其核心内容整理：

#### 一、`AMS`的核心功能
1. ​**Activity 生命周期管理**​
    - 控制 Activity 的启动、暂停、恢复、销毁等操作，通过 `ActivityStack` 和 `TaskRecord` 维护任务栈结构
    - 协调跨进程 Activity 启动（如 `startActivity()` 请求），触发目标进程的 `onCreate()` 等生命周期回调
2. ​**进程管理**​
    - 根据应用优先级（前台、后台、服务等）动态调整进程优先级（通过 `OOM_ADJ` 值），管理进程的创建、销毁和内存回收
    - 通过 `ProcessRecord` 记录进程状态，与 Zygote 进程协作 fork 新进程
3. ​**四大组件管理**​
    - ​**Activity**​：启动、切换、销毁。
    - ​**Service**​：启动、绑定、停止。
    - ​`BroadcastReceiver`​：注册、发送、接收广播。
    - ​`ContentProvider`：管理数据共享权限及访问
4. ​**任务栈与多窗口管理**​
    - 维护 `TaskStack` 和 `Back Stack`，支持多任务切换和返回键逻辑
    - 处理分屏、画中画等场景下的任务栈调整
5. ​`ANR`（应用无响应）监控**​
    - 检测主线程阻塞（默认超时 5 秒），触发 `ANR` 弹窗并记录日志（`traces.txt`）
6. ​**内存管理**​
    - 监控系统内存，回收低优先级进程（如空进程），通过 `onTrimMemory()` 回调通知应用释放资源
#### 二、`AMS`的启动流程
1. ​`SystemServer` 初始化**​
    - 系统启动时，`SystemServer` 创建 `AMS` 实例，并注册到 `ServiceManager`
    - 初始化 `ActivityStackSupervisor`、`ProcessRecord` 等核心数据结构
2. ​**与 Zygote 通信**​
    - 通过 Binder 通知 Zygote 进程 fork 应用进程，完成进程创建和初始化

#### 三、关键类与数据结构
| 类名                       | 作用                       | 源码路径                                                                                   |
| ------------------------ | ------------------------ | -------------------------------------------------------------------------------------- |
| `ActivityManagerService` | `AMS` 核心类，管理所有组件操作       | `frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java` |
| `ActivityStack`          | 管理任务栈，维护 Activity 的顺序和状态 | `.../am/ActivityStack.java`                                                            |
| `ProcessRecord`          | 记录进程信息（如 PID、UID、优先级）    | `.../am/ProcessRecord.java`                                                            |
| `ActivityRecord`         | 封装单个 Activity 的状态和生命周期信息 | `.../am/ActivityRecord.java`                                                           |
#### 四、`AMS`的交互流程（以Activity启动为例）
1. ​**客户端请求**​
    - 应用调用 `startActivity()`，通过 `ActivityManager` 转发请求至 `AMS`
2. ​`AMS` 处理逻辑**​
    - 检查权限和合法性。
    - 确定目标进程是否存在，若不存在则通过 Zygote 创建新进程
    - 通知目标进程创建 Activity 并回调生命周期方法（如 `onCreate()`）
3. ​**返回结果**​
    - 完成 Activity 启动后，`AMS` 更新任务栈并返回结果给客户端

#### 五、面试常见问题
1. ​`AMS` 如何管理进程优先级？​**​
    - 根据应用状态（前台、后台）调整优先级，低内存时优先回收低优先级进程
2. ​`AMS` 如何处理 `ANR`？​**​
    - 监控主线程响应时间，超时后触发 `ANR` 并记录日志
3. ​`AMS` 与 Zygote 的关系？​**​
    - `AMS` 通过 Binder 通知 Zygote fork 新进程，Zygote 负责预加载系统资源
4. ​`AMS` 如何实现跨进程通信？​**​
    - 基于 Binder 机制，通过 `ActivityManager` 接口与客户端交互

#### 六、源码与扩展学习
- ​**源码路径**​：
    - AMS 核心类：`frameworks/base/services/core/java/com/android/server/am/`
    - 任务栈管理：`ActivityStack` 和 `TaskRecord` 类
- ​**扩展阅读**​：
    - AMS 与 WMS（窗口管理）、PMS（包管理）的协作机制
    - Android 10+ 中 `ActivityTaskManagerService` 的职责分离
