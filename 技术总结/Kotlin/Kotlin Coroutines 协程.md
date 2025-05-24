---
创建时间: 2025-04-25 16:59:26
作者: wangxiaoming
tags:
  - Kotlin
  - Coroutines
  - 协程
---
#### 一、核心概念
1. ​**定义**​  
    Kotlin 协程（`Coroutines`）是一种轻量级并发编程模型，通过挂起（Suspend）和恢复（Resume）机制实现非阻塞异步操作，替代传统线程模型。
2. ​**核心特性**​
    - ​**轻量高效**​：协程基于线程池实现，可创建数百万个协程而无需担心资源耗尽
    - ​**挂起函数**​：通过 `suspend` 关键字标记，支持异步操作（如网络请求）的暂停与恢复，不阻塞线程
    - ​**结构化并发**​：通过作用域（`CoroutineScope`）管理协程生命周期，确保资源安全释放
#### 二、基础使用
##### 1）启动协程
```kotlin
//启动不带结果的协程  
val job = GlobalScope.launch {  }  
//启动带结果的协程  
val deferred = GlobalScope.async {  }  
val result = deferred.await()
```
##### 2）调度器（Dispatchers）

| 调度器                      | 适用场景      | 线程池特性            |
| ------------------------ | --------- | ---------------- |
| `Dispatchers.Main`       | UI 操作     | 单线程（Android 主线程） |
| `Dispatchers.IO`         | 文件/网络 I/O | 弹性线程池（默认 64 线程）  |
| `Dispatchers.Default`    | CPU 密集型计算 | 固定线程池（CPU 核数）    |
| `Dispatchers.Unconfined` | 无特定限制（慎用） | 随机线程             |

##### 3）挂起与恢复
```kotlin
/**  
 * 挂起与恢复  
 */  
suspend fun fetchData():String {  
    delay(1000)  
    return "Data"  
}
```
#### 三、高级特性
##### 1）Flow 流式处理
处理异步数据流，支持背压（`Backpressure`）和流式操作符（如 `map`, `filter`）：
```kotlin
/**  
 * Flow流式处理  
 */  
flow<Int> {  
    emit(1)  
    emit(2)  
}.map {   
it * 2  
}.collect {  
    println(it)  
}
```
##### 2）Channel 通信
协程间通过 `Channel` 传递数据，避免共享内存竞争
```kotlin
/**  
 * Channel通信  
 * 协程间通过 Channel 传递数据，避免共享内存竞争：  
 */  
val channel = Channel<Int>()  
GlobalScope.launch { channel.send(1) }  
GlobalScope.launch { println(channel.receive()) }
```
##### 3）Actor模型
通过 `actor` 封装状态和逻辑，解决并发问题：
```kotlin

```
##### 4）异常处理
- ​**取消传播**​：父协程取消时，子协程自动取消。
- `​SupervisorJob`​：隔离子协程异常，避免整体取消：
```kotlin
/**  
 * 异常处理  
 * 取消传播：父协程取消时，子协程自动取消。  
 * SupervisorJob：隔离子协程异常，避免整体取消：  
 */  
val supervisor = SupervisorJob()  
val scope = CoroutineScope(Dispatchers.Main + supervisor)
```
#### 四、原理与优化
##### 1）协程与线程优化
| ​**特性**​ | ​**协程**​     | ​**线程**​    |
| -------- | ------------ | ----------- |
| 调度方式     | 协作式挂起        | 抢占式调度       |
| 资源消耗     | 低（约 50KB/协程） | 高（约 1MB/线程） |
| 上下文切换    | 用户态切换（无内核开销） | 内核态切换（高延迟）  |
##### 2）性能优化
- **避免频繁切换上下文**​：减少 `withContext` 调用。
- ​**合理选择调度器**​：CPU 任务用 `Default`，I/O 任务用 `IO`。
- ​**取消无效协程**​：通过 `job.cancel()` 释放资源。

#### 五、面试考察点
1. **协程与线程的区别**​
    - 协程是用户态协作式调度，线程是内核态抢占式调度。
    - 协程通过挂起避免阻塞线程，提升资源利用率。
2. ​**`Flow` 与 `RxJava` 对比**​
    - `Flow` 基于协程，更轻量；`RxJava` 基于观察者模式，功能更复杂。
    - `Flow` 支持冷流（按需发射数据），`RxJava` 支持热流（主动推送数据）。
3. ​**协程取消机制**​
    - 通过 `Job.cancel()` 触发取消，需在挂起函数中检查 `isActive`。
    - `withTimeout` 设置超时自动取消。
#### 六、注意事项
1. ​**避免内存泄漏**​
    - 使用 `viewModelScope` 或 `lifecycleScope` 绑定生命周期。
    - 避免在 `GlobalScope` 中启动长期运行的协程。
2. ​**调试技巧**​
    - 启用协程调试模式：`-Dkotlinx.coroutines.debug`。
    - 通过 `Job` 的 `invokeOnCompletion` 监听状态。
