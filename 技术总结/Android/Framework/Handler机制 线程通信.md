---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - Android
  - Handler
---
#### 一、Handler是什么？
`Handler`是Android 中用于 **线程间通信** 的核心机制，负责在不同线程间传递和处理消息。其核心组件包括：
- `Message`:消息的载体，包含操作指令和数据
- `MessageQueue`:消息队列，存储待处理的消息
- `Looper`:消息循环器，负责从队列中取出消息并分发给Handler
- `Handler`：消息处理器，发送消息并处理消息逻辑

#### 二、Handler的原理
##### 1）消息驱动模型
- 发送消息：通过`sendMessage()`或`post(Runnable)`将消息加入队列
- 消息入队：消息按时间戳排序，延时消息插入到队列合适位置
- 轮询处理：`Looper`通过`loop()`方法循环调用`MessageQueue.next()`取出消息，分发给目标`Handler`的`handleMessage()`处理
##### 2）主线程的特殊性
- 主线程的`Looper`由系统自动创建（`ActivityThread.main()`中初始化），而子线程需手动调用`Looper.prepare()`和`Looper.loop()`
##### 3）同步屏障与异步消息
- 通过`postSyncBarrier()`插入同步屏障，优先处理异步消息（如`UI`绘制）

#### Handler的使用
##### 1）基本用法
- 主线程：直接创建`Handler`（默认绑定主线程`Looper`）
```java
todo
```
- 子线程：需手动初始化`Looper`
```java
todo
```
发送消息：
- `sendMessage(Message)`：发送消息对象
- `post(Runnable)`：发送`Runnable`任务

##### 2）`HandlerThread`
- 封装了`Looper`的子线程,简化使用
```java
todo
```
- 适用于需要后台持续处理任务的场景

#### 四、注意事项
##### 1）内存泄漏
- 问题：非静态内部类Handler持有外部类（如Activity）引用，导致无法回收
- 解决：使用静态内部类+弱引用(`WeakReference`)或`removeCallbacksAndMessage(null)` 及时清理消息
##### 2）线程安全
- 避免多线程同时操作Handler,必要时用`synchronized`同步
##### 3）耗时操作
- Handler的`handleMessage()`中避免执行耗时任务，否则会阻塞消息队列

#### 五、优化点
##### 1）复用Message
- 通过`Message.obtain()`复用消息对象，减少内存分配
##### 2）合理使用`IdleHandler`
- 在消息队列空闲时执行低优先级任务（如预加载数据）
##### 3）替代方案
- 使用`LiveData + ViewModel 或 RxJava`简化线程切换
##### 4）消息清理
- 在Activity/Fragment销毁时调用 `handler.removeCallbacksAndMessage(null)`

#### 六、面试常见考点
##### 1）机制原理
- 描述`Handler、Looper、MessageQueue`协作流程
##### 2）主线程`Looper`为何不阻塞
- Linux 的 `epoll`机制使 `MessageQueue.next()`在无消息时休眠，不占用CPU
##### 3）同步屏障与异步消息
- 用途及实现原理（如View绘制时优先处理异步消息）
##### 4）内部泄漏场景及解决
- 静态内部类+弱引用的必要性
##### 5）`HandlerThread` 与 `IntentService`
- 如何利用`HandlerThread`实现后台任务
#### 扩展思考
`Native`层实现：`MessageQueue` 的 `nativePollOnce`和`nativeWake()`通过`JNI`调用`Linux`的`epoll`实现阻塞和唤醒
`ANR`与`Handler`：主线程消息处理超时导致`ANR`，需避免在Handler中执行耗时操作