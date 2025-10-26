---
创建时间: "2025-06-24 11:38:43"
作者: wangxiaoming
tags:
---


在 Java 并发编程中，`LockSupport` 是一个关键的底层工具类，主要用于线程的**阻塞与唤醒**，是实现自定义锁、线程协作等高级功能的基础。以下是其核心考点的详细总结：

#### 一、核心方法与功能
`LockSupport` 的核心方法围绕线程的**阻塞（`park`）​**和**唤醒（`unpark`）​**展开，需重点掌握其方法签名、行为及区别：

|​**方法**​|​**描述**​|
|---|---|
|`static void park()`|阻塞当前线程，直到被 `unpark(Thread)` 唤醒或被中断（无许可时阻塞）。|
|`static void parkNanos(long nanos)`|阻塞当前线程，最多等待 `nanos` 纳秒（超时后自动唤醒）。|
|`static void parkUntil(long deadline)`|阻塞当前线程，直到指定时间 `deadline`（毫秒级时间戳）到达（超时后自动唤醒）。|
|`static void unpark(Thread thread)`|唤醒指定线程（使其从 `park()` 中恢复，即使未调用过 `park()` 也可提前发放许可）。|
#### 二、核心考点详解
##### 1. ​许可（Permit）机制​
`LockSupport` 的底层逻辑基于一个**许可（虚拟令牌）​**​：
- 每个线程默认无许可（`permit=0`）。
- 调用 `park()` 时，若许可为 `0`，线程阻塞；若许可为 `1`，许可减为 `0`，线程继续执行（不阻塞）。
- 调用 `unpark(thread)` 时，为目标线程发放许可（无论其是否有许可，许可变为 `1`）。
- 
​**关键特性**​：
- 许可是一次性的：`park()` 消耗许可，`unpark()` 发放许可（可重复发放）。
- `unpark()` 可在 `park()` 之前调用：此时许可已存在，后续 `park()` 不会阻塞（“提前唤醒”）。

##### 2. ​与 `wait()`/`notify()` 的对比​
`LockSupport` 与 `Object.wait()`/`Thread.notify()` 都可实现线程协作，但核心差异是**是否需要持有锁**​：

|​**维度**​|`LockSupport`|`Object.wait()`/`notify()`|
|---|---|---|
|​**锁依赖**​|无需持有锁（可直接调用）|必须在 `synchronized` 块中调用（否则抛 `IllegalMonitorStateException`）。|
|​**阻塞原因**​|无许可或超时|调用 `wait()` 后释放锁，等待 `notify()` 唤醒。|
|​**唤醒方式**​|显式调用 `unpark(Thread)`|调用 `notify()`/`notifyAll()`。|
|​**中断处理**​|被中断时 `park()` 返回（清除中断状态）|被中断时 `wait()` 抛出 `InterruptedException`。|
|​**灵活性**​|更轻量（无需锁），适合自定义同步器|依赖对象监视器，适合内置同步需求（如 `synchronized`）。|
##### 3. ​**超时与中断处理**​
- ​**超时机制**​：`parkNanos()` 和 `parkUntil()` 支持限时阻塞，避免无限等待（如防止死锁）。  
    示例：
```java
LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(3)); // 最多阻塞3秒
```
- ​**中断处理**​：`park()` 被中断时不会抛异常，而是**清除中断状态**并返回（需手动检查 `Thread.currentThread().isInterrupted()`）。  
    对比 `Thread.sleep()`：被中断时抛出 `InterruptedException`，中断状态会被清除。
##### 4. ​**实现自定义锁**​
`LockSupport` 是实现自定义锁的基础工具（如 `ReentrantLock` 的底层实现依赖它）。以下是简化的“非公平锁”实现逻辑：
```java
public class SimpleLock {
    private Thread owner = null; // 当前持有锁的线程

    public void lock() {
        Thread current = Thread.currentThread();
        while (owner != null || !compareAndSetOwner(null, current)) { // CAS 更新 owner
            LockSupport.park(); // 无许可则阻塞
        }
    }

    public void unlock() {
        Thread current = Thread.currentThread();
        if (owner != current) {
            throw new IllegalMonitorStateException("未持有锁");
        }
        owner = null;
        LockSupport.unpark(owner); // 唤醒等待线程（实际需维护等待队列）
    }

    // 简化的 CAS 操作（实际用 Unsafe 或 AtomicReference）
    private boolean compareAndSetOwner(Thread expect, Thread update) {
        // 实际实现依赖原子操作（如 AtomicReference）
        return true;
    }
}
```

**关键点**​：
- 自定义锁需维护等待队列（`LockSupport` 本身不管理队列，需自行实现）。
- `unpark()` 需唤醒队列中的下一个线程（而非随机唤醒）。
##### 5. ​**解决死锁问题**​
`LockSupport` 可用于检测和解决死锁（如通过超时机制避免无限等待）。例如：
```java
// 线程1：尝试获取锁A和锁B
synchronized (lockA) {
    if (!lockB.tryLock(1, TimeUnit.SECONDS)) { // 尝试加锁B，超时1秒
        LockSupport.parkNanos(TimeUnit.MILLISECONDS.toNanos(100)); // 短暂阻塞后重试
    }
}

// 线程2：尝试获取锁B和锁A（与线程1相反顺序）
synchronized (lockB) {
    if (!lockA.tryLock(1, TimeUnit.SECONDS)) {
        LockSupport.parkNanos(TimeUnit.MILLISECONDS.toNanos(100));
    }
}
```
#### 三、常见面试题
1. ​**`LockSupport.park()` 和 `Thread.sleep()` 的区别？​**​
    - `park()` 无需持有锁，可被中断（清除中断状态）；`sleep()` 需在同步块外调用，被中断时抛异常。
    - `park()` 基于许可机制，`sleep()` 是操作系统级阻塞。
2. ​**`unpark(Thread)` 可以在 `park()` 之前调用吗？​**​  
    可以。此时目标线程的许可被提前发放，后续调用 `park()` 不会阻塞（“提前唤醒”）。
    
3. ​**如何用 `LockSupport` 实现一个简单的线程协作？​**​  
    示例：生产者-消费者模型中，队列满时生产者 `park()`，消费者消费后 `unpark(生产者)`；队列空时消费者 `park()`，生产者生产后 `unpark(消费者)`。
    
4. ​**`LockSupport` 的底层原理是什么？​**​  
    基于 `sun.misc.Unsafe` 类的 `park()` 和 `unpark()` 方法，直接调用操作系统的线程调度接口（如 Linux 的 `futex`）。