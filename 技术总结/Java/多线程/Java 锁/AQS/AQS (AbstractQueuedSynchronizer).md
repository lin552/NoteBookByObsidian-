---
创建时间: 2025-04-18 20:35:34
作者: wangxiaoming
tags:
  - AQS
  - ReetrantLock
  - Semaphore
  - CountDownLatch
---
Java 中的 `​AQS（AbstractQueuedSynchronizer，抽象队列同步器）`​**​ 是 `java.util.concurrent.locks` 包下的核心同步框架，是 `ReentrantLock`、`CountDownLatch`、`Semaphore` 等并发工具类的底层实现基础。其核心设计思想是通过**状态变量（state）​**和**等待队列**管理多线程对共享资源的访问。以下是 `AQS` 的核心考点及详细解析：

#### 一、`AQS`的核心结构
`AQS` 是一个抽象类，定义了同步器的通用模板，核心包含两个关键组件：
##### **1. 状态变量（state）​**​
- ​**定义**​：`volatile int state`，表示同步状态（具体语义由子类定义）。
- ​**作用**​：
    - 在 `ReentrantLock` 中，`state` 表示锁的重入次数（0 表示未锁定，>0 表示已锁定且重入次数）。
    - 在 `CountDownLatch` 中，`state` 表示剩余需要等待的线程数（初始为 `count`，递减至 0 时释放所有等待线程）。
    - 在 `Semaphore` 中，`state` 表示可用的许可数量（初始为 `permits`，获取时递减，释放时递增）。
- ​**可见性保证**​：通过 `volatile` 修饰，确保多线程下的可见性。

##### ​**2. 等待队列（`CLH Queue`）​**​
- ​**定义**​：双向链表（`CLH` 锁的变种），用于管理竞争锁失败的线程（按 FIFO 顺序排队）。
- ​**节点（Node）结构**​：每个节点封装一个等待线程，包含以下关键字段：
    - `thread`：等待的线程引用。
    - `waitState`：等待状态（如 `0` 未等待、`WAITING` 等待中、`CANCELLED` 已取消）。
    - `prev` 和 `next`：前驱和后继节点指针（双向链表）。

#### 二、`AQS`的核心方法
`AQS` 定义了同步器的核心操作模板，子类通过重写这些方法实现具体同步逻辑：
##### **1. 获取锁（acquire）​**​
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) {
        selfInterrupt(); // 自我中断（可选）
    }
}
```
- ​**流程**​：
    1. ​**尝试获取（`tryAcquire`）​**​：子类实现，判断是否可获取锁（如 `ReentrantLock` 检查 `state` 是否可用且当前线程是否已持有锁）。
    2. ​**加入等待队列（`addWaiter`）​**​：若获取失败，创建新节点并加入等待队列尾部。
    3. ​**阻塞等待（`acquireQueued`）​**​：循环检查前驱节点是否为头节点（或是否被唤醒），若未获取到锁则调用 `park()` 阻塞当前线程。

##### ​**2. 释放锁（release）​**​
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitState != 0) {
            unparkSuccessor(h); // 唤醒后继节点
        }
        return true;
    }
    return false;
}
```
- ​**流程**​：
    1. ​**尝试释放（`tryRelease`）​**​：子类实现，判断是否可释放锁（如 `ReentrantLock` 检查 `state` 是否递减为 0）。
    2. ​**唤醒后继（`unparkSuccessor`）​**​：若释放成功且队列头节点等待状态非 0，唤醒头节点的后继线程（使其重新竞争锁）。
#### ​**3. 其他辅助方法**​
- `addWaiter(Node mode)`：将当前线程封装为节点并加入等待队列。
- `acquireQueued(Node node, int arg)`：阻塞当前线程，直到被前驱节点唤醒或获取到锁。
- `cancelAcquire(Node node)`：取消节点的等待（如线程中断或超时）。

#### 三、`AQS`的两种模式
`AQS` 支持**独占模式（Exclusive）​**和**共享模式（Shared）​**，区别在于锁的持有方式：
#### ​**1. 独占模式（默认）​**​
- ​**定义**​：锁仅能被一个线程持有（如 `ReentrantLock`）。
- ​**实现**​：节点模式为 `Node.EXCLUSIVE`，释放锁时仅唤醒后继节点（下一个竞争者）。
#### ​**2. 共享模式**​
- ​**定义**​：锁可被多个线程共享（如 `CountDownLatch`、`Semaphore`）。
- ​**实现**​：节点模式为 `Node.SHARED`，释放锁时需唤醒所有后续节点（或批量唤醒）。
#### ​四、`AQS` 在并发工具中的应用示例
##### 1. `ReentrantLock`（独占模式）​​
- ​**state**​：表示锁的重入次数（0 → 未锁定；>0 → 已锁定，值为重入次数）。
- ​`tryAcquire`**​：若当前线程已持有锁（`state > 0` 且线程等于 `exclusiveOwnerThread`），则 `state++`（重入）；否则尝试获取锁（`state == 0` 时设置为 1 并标记当前线程为持有者）。
- ​`tryRelease`**​：若当前线程是持有者，`state--`；若 `state == 0`，则释放锁并清空持有线程。
##### ​**2. `CountDownLatch`（共享模式）​​
- ​**state**​：表示剩余需要等待的线程数（初始为 `count`，递减至 0 时释放所有线程）。
- ​`tryAcquireShared`：若 `state == 0`，返回 `1`（表示可获取）；否则返回 `-1`（表示需等待）。
- ​`tryReleaseShared`：`state--`（需 `CAS` 保证原子性），若 `state == 0` 则返回 `true`（唤醒所有等待线程）。
#### 五、`AQS`的关键特性与设计思想
##### ​1. 无锁编程与 `CAS​`
`AQS` 的 `state` 修改和队列操作依赖 `CAS`（如 `compareAndSetState`），确保多线程下的原子性。例如，`ReentrantLock` 的非公平锁通过`CAS` 尝试更新 `state` 来获取锁。
##### ​2. 等待队列的管理​
通过双向链表维护等待线程，确保公平性（公平锁按队列顺序唤醒，非公平锁允许插队）。例如，`ReentrantLock` 的公平锁在 `acquire` 时会先检查队列是否有前驱节点，若有则排队。
##### ​3. 可扩展性​
`AQS` 是抽象类，子类通过重写 `tryAcquire`、`tryRelease` 等方法实现不同同步逻辑，体现了“模板方法模式”。

#### 六、常见面试题
1. **​`AQS` 的核心作用是什么？
    `AQS` 是同步器的抽象框架，通过状态变量（state）和等待队列管理多线程对共享资源的访问，是 `ReentrantLock`、`CountDownLatch` 等工具的底层实现基础。
    
2. ​`AQS` 的等待队列是什么结构？有什么作用？​
    双向链表（`CLH` 队列变种），用于管理竞争锁失败的线程，按 FIFO 顺序排队等待唤醒。
    
3. ​`AQS` 的 state 变量的语义由谁定义？​**​  
    由子类定义（如 `ReentrantLock` 中表示重入次数，`CountDownLatch` 中表示剩余计数）。
    
4. ​`AQS` 支持哪两种模式？分别举例说明。​**​  
    独占模式（如 `ReentrantLock`，仅一个线程持有锁）和共享模式（如 `CountDownLatch`，多个线程共享锁）。
    
5. ​`AQS` 的 acquire 和 release 方法的核心流程是什么？​**​  
    `acquire`：尝试获取锁 → 失败则加入队列并阻塞；`release`：尝试释放锁 → 成功则唤醒后继节点。
    
6. ​**公平锁与非公平锁在 `AQS` 中的实现差异是什么？​**​  
    公平锁在 `acquire` 时会检查等待队列是否有前驱节点（避免插队），非公平锁直接尝试获取锁（允许插队）。