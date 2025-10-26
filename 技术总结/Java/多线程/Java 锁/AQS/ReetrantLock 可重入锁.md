---
创建时间: 2025-04-18 22:15:55
作者: wangxiaoming
tags:
  - ReetrantLock
  - AQS
---
Java 中的 `ReentrantLock` 是 `java.util.concurrent.locks` 包下的可重入互斥锁，提供了比 `synchronized` 更灵活的同步机制（如可中断锁、尝试锁、超时锁、公平锁策略等）。其核心考点围绕**底层原理**​（`AQS`）、**功能特性**​（公平/非公平、可重入）、**与 `synchronized` 的对比**及**使用场景**展开。以下是详细梳理：
#### 一、核心特性
`ReentrantLock` 的核心优势在于**灵活的锁控制**，主要特性包括：
##### 1. 可重入性（`Reentrancy`）​​
同一线程可多次获取同一把锁（计数器递增），避免自身死锁。例如：
```java
public class ReentrantDemo {
    private ReentrantLock lock = new ReentrantLock();

    public void outer() {
        lock.lock();
        try {
            inner(); // 内部调用同样被锁保护的方法
        } finally {
            lock.unlock();
        }
    }

    public void inner() {
        lock.lock();
        try {
            // 操作共享资源...
        } finally {
            lock.unlock();
        }
    }
}
```
- ​**实现**​：通过状态变量 `state` 记录锁的重入次数（`state` 初始为 0，每获取一次锁加 1，释放时减 1；`state == 0` 时锁真正释放）。
##### ​2. 公平与非公平锁​
- ​**公平锁**​：锁的获取按等待队列的顺序分配（先等待的线程优先获取），避免线程饥饿。
- ​**非公平锁**​：允许新来的线程插队（直接尝试获取锁，成功则抢占），吞吐量更高但可能导致饥饿。
- ​**默认策略**​：`ReentrantLock` 构造函数默认创建非公平锁（`new ReentrantLock()` 等价于 `new ReentrantLock(false)`）；通过 `new ReentrantLock(true)` 可指定公平锁。
##### ​3. 可中断锁获取​
支持在等待锁的过程中响应中断（`lockInterruptibly()` 方法），避免无限等待。
```java
public void method() throws InterruptedException {
    lock.lockInterruptibly(); // 可中断的锁获取
    try {
        // 操作共享资源...
    } finally {
        lock.unlock();
    }
}
```
##### ​4. 尝试非阻塞锁获取​
支持尝试获取锁（`tryLock()`），若锁不可用则立即返回 `false`，避免阻塞。
```java
if (lock.tryLock()) { // 尝试获取锁（无参版本）
    try {
        // 操作共享资源...
    } finally {
        lock.unlock();
    }
} else {
    // 锁被占用，执行其他逻辑...
}
```
- ​**带超时版本**​：`tryLock(long timeout, TimeUnit unit)` 支持指定超时时间，超时后自动放弃。
##### ​**5. 条件变量（Condition）​**​
通过 `newCondition()` 方法创建多个 `Condition` 对象，实现更细粒度的线程等待/唤醒（替代 `synchronized` 的 `wait/notify`）。
```java
public class ProducerConsumer {
    private ReentrantLock lock = new ReentrantLock();
    private Condition notEmpty = lock.newCondition(); // 非空条件
    private Condition notFull = lock.newCondition();  // 未满条件
    private Queue<String> queue = new LinkedList<>();
    private int capacity = 10;

    public void produce() throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await(); // 等待队列未满
            }
            queue.add("item");
            notEmpty.signal(); // 唤醒等待非空的线程
        } finally {
            lock.unlock();
        }
    }

    public void consume() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await(); // 等待队列非空
            }
            String item = queue.poll();
            notFull.signal(); // 唤醒等待未满的线程
        } finally {
            lock.unlock();
        }
    }
}
```
#### 二、底层实现：`AQS(AbstractQueuedSynchronizer)`
`ReentrantLock` 的核心同步机制基于 `AQS`（抽象队列同步器），它定义了锁的获取、释放模板，并维护了一个**状态变量（state）​**和**等待队列**​（双向链表）。
##### 1. `AQS` 的核心结构​
- ​**状态变量（state）​**​：`volatile int state`，表示同步状态（如 `ReentrantLock` 中 `state` 表示锁的重入次数）。
- ​**等待队列**​：`CLHQueue`（`CLH` 锁的变种），用于管理竞争锁失败的线程（按 FIFO 顺序排队）。
- ​**独占模式（Exclusive）​**​：`ReentrantLock` 是独占锁（仅一个线程持有锁），`AQS` 支持共享模式（多个线程共享锁，如 `CountDownLatch`）。
##### ​**2. 锁的获取与释放流程**​
- ​**获取锁（acquire）​**​：
    1. 尝试通过 `tryAcquire(int arg)` 获取锁（`ReentrantLock` 中检查 `state` 是否可用，且当前线程是否已持有锁）。
    2. 若获取失败，将当前线程加入等待队列，并通过 `park()` 阻塞线程（自旋优化：`JDK 6` 后引入轻量级锁，减少阻塞）。
- ​**释放锁（release）​**​：
    1. 通过 `tryRelease(int arg)` 释放锁（`ReentrantLock` 中 `state` 减 1，若 `state == 0` 则真正释放锁）。
    2. 唤醒等待队列中的下一个线程（`unpark(Thread next)`）。

#### ​**三、公平与非公平锁的实现差异**
`ReentrantLock` 的公平与非公平策略通过 `tryAcquire` 方法的不同实现区分：
##### ​1. 非公平锁（默认）​​
- ​**逻辑**​：线程尝试直接获取锁（`state` 可用则获取），无需检查等待队列。
- ​**优点**​：减少线程切换，吞吐量更高（适合竞争激烈的场景）。
- ​**缺点**​：可能导致等待队列中的线程长期无法获取锁（饥饿）。
##### ​2. 公平锁​
- ​**逻辑**​：线程获取锁前需检查等待队列是否有前驱节点（若有则排队）。
- ​**优点**​：保证线程按等待顺序获取锁，避免饥饿。
- ​**缺点**​：频繁的队列检查增加了开销，吞吐量较低（适合竞争温和的场景）。
#### 四、与`synchronized`的对比
| ​**特性**​ | ​**ReentrantLock**​     | ​**synchronized**​     |
| -------- | ----------------------- | ---------------------- |
| 锁类型      | JDK 层面实现（基于 AQS）        | `JVM` 层面实现（Monitor 锁）  |
| 灵活性      | 支持可中断、尝试锁、超时锁、公平锁       | 固定（非公平、不可中断、无超时）       |
| 条件变量     | 支持多个 `Condition`（细粒度唤醒） | 仅一个等待队列（`wait/notify`） |
| 可重入性     | 是（通过 `state` 计数）        | 是（通过 Monitor 锁计数）      |
| 性能       | 早期差（AQS 实现复杂），后期接近      | 早期优（Monitor 锁），后期接近    |
| 使用复杂度    | 高（需手动释放锁）               | 低（自动释放）                |
#### 五、常见面试题
1. ​`ReentrantLock` 的核心特性有哪些？​**​  
    可重入性、公平/非公平策略、可中断锁获取、尝试锁、超时锁、多条件变量（`Condition`）。
    
2. ​`ReentrantLock` 如何实现可重入？​**​  
    通过状态变量 `state` 记录锁的重入次数：当前线程首次获取锁时 `state=1`，重入时 `state++`，释放时 `state--`（`state=0` 时真正释放锁）。
    
3. ​**公平锁与非公平锁的区别是什么？如何选择？​**​  
    公平锁按等待队列顺序分配锁（避免饥饿），非公平锁允许插队（吞吐量高）。竞争激烈时选公平锁，否则选非公平锁。
    
4. ​`ReentrantLock` 的底层实现依赖什么？​**​  
    依赖 `AQS`（`AbstractQueuedSynchronizer`），通过 `state` 状态变量和等待队列管理锁的获取与释放。
    
5. ​`ReentrantLock` 为什么需要手动释放锁？​**​  
    因为 `lock()` 方法没有自动释放机制（区别于 `synchronized` 的隐式释放），需在 `finally` 块中调用 `unlock()` 避免死锁。
    
6. ​`ReentrantLock` 的 `tryLock()` 和 `lock()` 有什么区别？​**​  
    `tryLock()` 尝试获取锁（失败立即返回），`lock()` 阻塞直到获取锁；`tryLock(long, TimeUnit)` 支持超时机制。
    
7. ​**如何用 `ReentrantLock` 实现生产者-消费者模型？​**​  
    使用 `Condition` 的 `await()` 和 `signal()` 方法替代 `wait/notify`，实现更细粒度的等待/唤醒（如区分“队列满”和“队列空”条件）。