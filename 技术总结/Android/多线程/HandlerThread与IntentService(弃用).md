---
创建时间: 2025-05-07 10:09:20
作者: wangxiaoming
tags:
  - Android
  - handlerThread
  - IntentService
---
#### 一、`HandlerThread`
##### 1）解决的问题
- **背景**​：在 Android 中，子线程无法直接更新 `UI`，而主线程（`UI` 线程）又不能执行耗时操作（如网络请求、文件读写）。传统方式需要在子线程中手动管理任务队列和线程生命周期，代码复杂且易出错。
- ​**目标**​：简化子线程任务管理，提供串行执行任务的机制，避免频繁创建/销毁线程的开销。
##### 2）原理
- **本质**​：继承自 `Thread`，内部封装了 `Looper` 和 `MessageQueue`，通过消息循环机制实现任务串行处理。
- ​**关键源码**​：
```java
// HandlerThread 的 run 方法
public void run() {
    Looper.prepare();  // 创建 Looper
    synchronized (this) {
        mLooper = Looper.myLooper();
        notifyAll();   // 通知等待的线程
    }
    Looper.loop();     // 消息循环
    mLooper = null;
}
```
- ​**任务提交**​：通过 `Handler` 将任务（`Message` 或 `Runnable`）发送到 `HandlerThread` 的消息队列中，按顺序执行。
##### 3）使用方式
```java
// 创建并启动 HandlerThread
HandlerThread handlerThread = new HandlerThread("BackgroundThread");
handlerThread.start();

// 获取 Looper 并创建 Handler
Handler handler = new Handler(handlerThread.getLooper()) {
    @Override
    public void handleMessage(Message msg) {
        // 执行耗时任务
    }
};

// 提交任务
handler.post(() -> {
    // 耗时操作（如网络请求）
});
```
#### 4）注意事项
- ​**线程生命周期**​：任务完成后需调用 `quit()` 或 `quitSafely()` 终止线程，避免内存泄漏。
- ​**任务顺序**​：任务按提交顺序串行执行，需避免长时间任务阻塞后续操作。
- ​**与主线程通信**​：若需更新 `UI`，需通过主线程的 `Handler` 切换上下文。
##### 5）优化点
- ​**任务拆分**​：将大任务拆分为多个小任务，避免单次执行时间过长。
- ​**优先级设置**​：通过 `setPriority()` 调整线程优先级，优化资源分配。
- ​**复用线程**​：避免频繁创建/销毁 `HandlerThread`，复用已有实例。
##### 6）面试考察点
- ​**与普通 Thread 的区别**​：`HandlerThread` 内置消息循环，支持任务队列管理。
- ​**如何保证任务顺序执行**​：依赖 `MessageQueue` 的 FIFO 特性。
- ​**适用场景**​：需串行处理后台任务的场景（如图片加载队列）。

#### 二、`IntentService`(弃用) -> 推荐`JobIntentService`
##### 1）解决的问题
- **背景**​：`Service` 默认运行在主线程，无法直接处理耗时操作；传统方式需手动创建线程并管理生命周期。
- ​**目标**​：简化后台服务的线程管理，自动处理任务队列，任务完成后自动销毁。
##### 2）原理
- ​**本质**​：继承自 `Service`，内部封装了 `HandlerThread` 和 `Handler`，通过 `onHandleIntent()` 处理任务。
- ​**关键源码**​：
```java
// IntentService 的 onCreate 方法
@Override
public void onCreate() {
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();
    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}

// 任务处理
private final class ServiceHandler extends Handler {
    public void handleMessage(Message msg) {
        onHandleIntent((Intent) msg.obj);  // 执行耗时任务
        stopSelf(msg.arg1);                // 任务完成后停止服务
    }
}
```
- **任务提交**​：通过 `startService(Intent)` 发送任务，按顺序加入队列执行。
##### 3）使用方式
```java
public class MyIntentService extends IntentService {
   public MyIntentService() {
       super("MyIntentService");
   }

   @Override
   protected void onHandeIntent(@Nullable Intent intent){
       //执行耗时任务（如下载文件）
   }
}

//启动服务
Intent intent = new Intent(context,MyIntentService.class)
context.startService(intent);
```
##### 4）注意事项
- ​**单线程模型**​：所有任务串行执行，需避免并发问题。
- ​**自动销毁**​：任务完成后自动调用 `stopSelf()`，无需手动管理。
- ​**与 Activity 通信**​：通过 `BroadcastReceiver` 或 `LiveData` 回调结果，避免直接操作 `UI`。
##### 5）优化点
- ​**任务拆分**​：复杂任务拆分为多个 `Intent` 分批次处理。
- ​**并发需求**​：若需并行处理，可结合 `ThreadPoolExecutor` 或 `WorkManager`。
- ​**优先级控制**​：通过 `Intent` 传递参数，动态调整任务优先级。
#### 三、对比与适用场景
|​**特性**​|​**HandlerThread**​|​**IntentService**​|
|---|---|---|
|​**线程类型**​|普通子线程|封装了 `HandlerThread` 的 Service|
|​**任务管理**​|需手动提交任务到 `Handler`|自动通过 `Intent` 接收任务|
|​**生命周期**​|需手动启动/停止|任务完成后自动销毁|
|​**适用场景**​|需要长期运行的后台任务（如定时同步）|需要后台处理多个任务的请求（如文件上传）|
|​**通信方式**​|通过 `Handler` 与主线程交互|通过 `BroadcastReceiver` 或回调|
#### 四、总结
- ​**`HandlerThread`**​：适合需要**串行处理后台任务**且需与主线程通信的场景（如轮询、延迟任务）。
- ​**`IntentService`**​：适合需要**后台服务化处理多个任务**的场景（如批量下载、数据同步）。
- ​**核心设计思想**​：两者均通过 `HandlerThread` 实现任务队列管理，简化多线程编程，但需注意任务顺序和资源释放。

**面试高频问题**​：
1. `HandlerThread` 如何保证任务顺序执行？
2. `IntentService` 为什么能自动停止？
3. 两者与 `AsyncTask`、`RxJava` 的区别？
4. 如何优化长时间任务阻塞问题？