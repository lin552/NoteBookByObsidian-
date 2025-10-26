---
创建时间: "2025-08-20 15:05:23"
作者: wangxiaoming
tags:
---
在 Android 中，​**`Looper`不能通过构造方法直接创建**​（无论是公开还是反射方式），其设计初衷是通过静态方法（如 `prepare()`、`prepareMainLooper()`）配合 `loop()`启动消息循环。这一限制与 `Looper`的内部机制、线程绑定关系及消息队列的安全初始化密切相关。

#### 1.`Looper`的构造方法是私有的
```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed); // 创建消息队列
    mThread = Thread.currentThread();      // 绑定当前线程
}
```
构造方法的私有性直接阻止了外部通过 `new Looper()`的方式创建实例。开发者必须通过静态方法 `prepare()`或 `prepareMainLooper()`间接创建 `Looper`。

#### 2.为什么禁止直接通过构造方法创建？
`Looper`的核心职责是为线程提供**消息循环能力**，其创建过程需要严格的初始化检查和线程绑定逻辑。直接调用构造方法会绕过这些关键步骤，导致以下问题：
##### （1）线程重复绑定风险
每个线程最多只能有一个 `Looper`（否则会抛出 `RuntimeException: Only one Looper may be created per thread`）。`prepare()`方法会先检查当前线程是否已存在 `Looper`（通过 `sThreadLocal.get()`），若存在则拒绝创建；而直接调用构造方法会跳过这一检查，导致线程中出现多个 `Looper`，破坏消息循环的唯一性。
###### a.如何保证一个线程一个`Looper`
1. `ThreadLocal`存储：为每个线程独立保存其 `Looper`实例，确保线程间隔离。
2. ​**创建前的存在性检查**​：在 `Looper.prepare()`方法中，强制检查当前线程是否已存在 `Looper`，若存在则直接抛出异常。

`Looper`内部通过一个静态的 `ThreadLocal<Looper>`实例（`sThreadLocal`）来存储每个线程的 `Looper`对象：
```java
// Looper 源码（API 33）
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
```

`Looper.prepare()`是创建 `Looper`的入口方法（主线程的 `prepareMainLooper()`本质也是调用 `prepare(true)`），其核心逻辑如下：
```java
// Looper.prepare() 源码（简化版）
public static void prepare() {
    prepare(true); // 默认允许退出（非主线程）
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) { 
        // 关键检查：当前线程的 ThreadLocal 中已有 Looper
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed)); // 不存在则创建并存储
}
```

##### （2）消息队列（`MessageQueue`）的关联缺失
`Looper`必须与线程的 `MessageQueue`绑定才能工作。`prepare()`方法在创建 `Looper`时，会通过 `nativeInit()`本地方法为线程关联一个 `MessageQueue`（底层由 C++ 实现）；而直接调用构造方法仅会创建 `MessageQueue`对象（`mQueue = new MessageQueue(...)`），但未完成线程与队列的底层绑定，导致消息无法正常分发。
##### (3)主线程的特殊限制
主线程（`UI` 线程）的 `Looper`由 `prepareMainLooper()`创建，其内部调用 `prepare(true)`并额外标记为主线程 `Looper`（`mIsMainThreadLooper = true`）。主线程 `Looper`禁止通过 `quit()`或 `quitSafely()`终止（否则会导致 `UI` 崩溃），而直接调用构造方法无法保证这一限制，可能引发不可预测的系统行为。

#### 3.正确创建`Looper`的方式
`Looper`的创建必须通过以下静态方法，以确保线程安全和功能完整性：
##### (1)子线程: `Looper.prepare() + Looper.loop()`
子线程需要手动创建 `Looper`，流程如下：
```java
new Thread(() -> {
    Looper.prepare(); // 1. 检查当前线程是否已有 Looper，无则创建并绑定
    Looper looper = Looper.myLooper(); // 2. 获取当前线程的 Looper
    Handler handler = new Handler(looper); // 3. 绑定 Looper 到 Handler
    Looper.loop(); // 4. 启动消息循环（阻塞等待消息）
}).start();
```
- `Looper.prepare()`：内部调用私有构造方法创建 `Looper`，并完成线程与 `MessageQueue`的绑定。
- `Looper.loop()`：启动消息循环，不断从 `MessageQueue`中取出消息并分发

##### (2)主线程：`Looper.prepareMainLooper()`
主线程的 `Looper`在应用启动时由 `ActivityThread.main()`调用 `Looper.prepareMainLooper()`创建，无需手动干预：
```java
// ActivityThread.main() 中的关键代码
Looper.prepareMainLooper(); // 主线程专用，禁止退出
ActivityThread thread = new ActivityThread();
thread.attach(false);
Looper.loop(); // 启动主线程消息循环
```
`prepareMainLooper()`内部调用 `prepare(true)`，创建主线程 `Looper`并标记为不可退出（`quitAllowed = false`）。

#### 4.强制反射创建的后果
即使通过反射绕过私有构造方法（如 `Looper.class.getDeclaredConstructor(boolean.class).newInstance(true)`），也会导致以下问题：
- ​**线程状态不一致**​：未通过 `prepare()`注册到 `ThreadLocal`，`Looper.myLooper()`无法获取到实例。
- ​**消息循环失效**​：未完成 `nativeInit()`本地绑定，`MessageQueue`无法接收或分发消息。
- ​**崩溃或 `ANR`**​：主线程若错误创建 `Looper`，可能导致 `UI` 事件无法处理（如触摸无响应）；子线程若未正确绑定，`Handler`发送的消息会丢失。

#### 总结
`Looper`禁止通过构造方法直接创建的核心原因是：其初始化过程需要严格的线程检查、`MessageQueue`绑定和主线程特殊逻辑，这些必须通过静态方法 `prepare()`/`prepareMainLooper()`完成。直接调用构造方法会破坏 `Looper`与线程的绑定关系，导致消息循环失效或系统崩溃。开发者应始终使用官方提供的标准流程创建 `Looper`。