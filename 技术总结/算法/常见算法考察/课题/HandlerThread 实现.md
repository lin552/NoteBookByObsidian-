---
创建时间: "2025-08-21 18:51:07"
作者: wangxiaoming
tags:
---
#### 一、核心原理

`HandlerThread`的本质是一个 `Thread`，其核心逻辑是在 `run()`方法中**初始化 `Looper` 并启动消息循环。具体流程如下：
1. 继承 `Thread`，重写 `run()`方法。
2. 在 `run()`中调用 `Looper.prepare()`创建当前线程的 `Looper`。
3. 启动消息循环 `Looper.loop()`，使线程持续处理消息队列中的任务。
4. 提供 `getLooper()`方法，返回当前线程的 `Looper`，用于创建关联的 `Handler`。
#### 二、代码实现
##### 1.手动实现简化版`HandlerThread`
```java
public class MyHandlerThread extends Thread {
    private Looper mLooper;

    @Override
    public void run() {
        super.run();
        // 1. 初始化当前线程的 Looper
        Looper.prepare(); 
        
        // 2. 获取当前线程的 Looper（供外部使用）
        mLooper = Looper.myLooper();
        
        // 3. 启动消息循环（阻塞当前线程，持续处理消息队列）
        Looper.loop();
    }

    // 提供方法获取 Looper
    public Looper getLooper() {
        return mLooper;
    }

    // 终止消息循环（释放资源）
    public void quit() {
        if (mLooper != null) {
            mLooper.quit();
        }
    }
}
```

##### 2.使用手写板`HandlerThread`
```java
// 创建自定义 HandlerThread
MyHandlerThread handlerThread = new MyHandlerThread();
handlerThread.start(); // 启动线程（内部会初始化 Looper 并启动循环）

// 创建关联该 Looper 的 Handler（用于发送/处理消息）
Handler handler = new Handler(handlerThread.getLooper()) {
    @Override
    public void handleMessage(@NonNull Message msg) {
        super.handleMessage(msg);
        // 在此处理消息（运行在 handlerThread 线程中）
        Log.d("HandlerThread", "处理消息: " + msg.what + "，线程：" + Thread.currentThread().getName());
    }
};

// 发送一条延迟消息（2秒后处理）
Message message = Message.obtain();
message.what = 100;
handler.sendMessageDelayed(message, 2000);

// 最终记得退出 Looper（避免内存泄漏）
@Override
protected void onDestroy() {
    super.onDestroy();
    handlerThread.quit(); // 终止消息循环
    handlerThread.interrupt(); // 可选：中断线程（如果需要立即停止）
}
```

#### 三、Android官方`HandlerThread`的实现
Android 官方的 `HandlerThread`内部已经封装了上述逻辑，无需手动实现。以下是其核心源码（简化版）：
##### 1.`HandlerThread`源码核心逻辑
```java
public class HandlerThread extends Thread {
    private int mPriority; // 线程优先级
    private int mTid = -1; // 线程 ID
    private Looper mLooper; // 当前线程的 Looper

    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT; // 默认优先级
    }

    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }

    @Override
    protected void onLooperPrepared() {
        // 子类可重写此方法，在 Looper 准备完成后执行初始化操作
    }

    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare(); // 初始化当前线程的 Looper
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll(); // 通知等待获取 Looper 的线程
        }
        Process.setThreadPriority(mPriority); // 设置线程优先级
        onLooperPrepared(); // 触发子类初始化
        Looper.loop(); // 启动消息循环
        mTid = -1;
    }

    // 获取当前线程的 Looper（阻塞直到 Looper 初始化完成）
    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait(); // 等待 run() 中初始化 Looper 并通知
                } catch (InterruptedException e) {
                    // 忽略中断异常
                }
            }
        }
        return mLooper;
    }

    // 终止消息循环
    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }

    // 安全终止消息循环（不处理已发送但未处理的消息）
    public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

    // 获取当前线程 ID
    public int getThreadId() {
        return mTid;
    }
}
```

#### 四、官方`HandlerThread`的使用示例
```java
// 1. 创建 HandlerThread（指定线程名称和优先级）
HandlerThread handlerThread = new HandlerThread("MyBackgroundThread", Process.THREAD_PRIORITY_BACKGROUND);
handlerThread.start(); // 启动线程（内部初始化 Looper 并启动循环）

// 2. 创建关联该 Looper 的 Handler
Handler backgroundHandler = new Handler(handlerThread.getLooper()) {
    @Override
    public void handleMessage(@NonNull Message msg) {
        super.handleMessage(msg);
        // 在此处理消息（运行在 MyBackgroundThread 线程中）
        Log.d("HandlerThread", "处理消息: " + msg.what + "，线程：" + Thread.currentThread().getName());
    }
};

// 3. 发送任务（延迟 2 秒执行）
backgroundHandler.postDelayed(() -> {
    Log.d("HandlerThread", "延迟任务执行，线程：" + Thread.currentThread().getName());
}, 2000);

// 4. 发送一次性消息
Message message = Message.obtain();
message.what = 200;
backgroundHandler.sendMessage(message);

// 5. 最终释放资源（避免内存泄漏）
@Override
protected void onDestroy() {
    super.onDestroy();
    // 终止消息循环（quit() 会丢弃未处理的消息；quitSafely() 会处理完已接收的消息）
    handlerThread.quitSafely(); 
    // 可选：中断线程（如果需要立即停止）
    handlerThread.interrupt();
}
```

#### 五、关键方法说明
|方法|说明|
|---|---|
|`getLooper()`|获取当前线程的 `Looper`（阻塞直到 `Looper`初始化完成）。|
|`quit()`|终止 `Looper`的消息循环（丢弃消息队列中未处理的消息）。|
|`quitSafely()`|安全终止 `Looper`（处理完已接收但未处理的消息后再退出）。|
|`post(Runnable)`|发送一个 `Runnable`到消息队列（等同于 `sendMessage(Message.obtain(...))`）。|
|`sendMessage(Message)`|发送消息到消息队列（通过 `Handler`的 `handleMessage`处理）。|
#### 六、适用场景
`HandlerThread`适用于需要在**子线程中处理异步任务**的场景，典型场景包括：
1. ​**后台耗时操作**​：如文件读写、网络请求（需配合其他组件，如 `OkHttp`本身支持异步，此处仅作示例）。
2. ​**延迟/周期性任务**​：如定时刷新数据（替代 `Timer`或 `AlarmManager`）。
3. ​**线程间通信**​：通过 `Handler`在子线程与主线程（或其他线程）间传递消息。

#### 七、注意事项
1. **线程生命周期管理**​：必须在 `Activity/Fragment`销毁时调用 `quit()`或 `quitSafely()`终止 `Looper`，否则线程可能无法退出，导致内存泄漏。
2. ​**避免阻塞 `Looper`**​：`Looper.loop()`是阻塞方法，若消息处理逻辑耗时过长（如大量计算），会导致后续消息无法及时处理，需将耗时操作进一步拆分到其他线程。
3. ​**线程优先级**​：默认优先级为 `PROCESS_PRIORITY_DEFAULT`，可根据需求调整（如后台线程设置为 `PROCESS_PRIORITY_BACKGROUND`降低 CPU 占用）。
4. ​**与主线程通信**​：若需在 `Handler`中更新 `UI`，需通过 `Handler(Looper.getMainLooper())`切换回主线程的 `Looper`。