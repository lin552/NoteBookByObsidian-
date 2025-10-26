---
创建时间: 2025-06-05 22:32:18
作者: wangxiaoming
tags:
  - CountDownLatch
  - AQS
---

Java 中的 ​**`CountDownLatch`**​ 是 `java.util.concurrent` 包下的同步辅助类，用于协调多个线程的执行顺序，允许一个或多个线程等待其他线程完成特定操作后再继续执行。其核心是通过**计数器**实现线程间的等待/通知机制。以下是其核心考点的详细梳理：

#### 一、核心功能与设计思想
`CountDownLatch` 的核心是一个**可递减的计数器**，初始化时设定一个正整数值（`count`）。其他线程完成任务后调用 `countDown()` 方法递减计数器；等待线程通过 `await()` 方法阻塞，直到计数器归零（所有任务完成）。

##### **关键特性**​：
- ​**一次性**​：计数器归零后无法重置（若需重复使用，需结合其他机制如 `CyclicBarrier`）。
- ​**多线程协作**​：支持多个线程等待（`await()` 可被多个线程调用），或单个线程等待多个任务。

#### 二、底层实现：基于`AQS`
`CountDownLatch` 底层通过 ​**`AbstractQueuedSynchronizer (AQS)`**​ 实现同步逻辑，核心依赖 `AQS` 的**状态变量（state）​**和**等待队列**。
#### ​**1. 状态变量（state）​**​
- `state` 表示剩余需要等待的任务数（初始化为 `count`）。
- 每次调用 `countDown()` 时，通过 `CAS` 递减 `state`（原子操作保证线程安全）。
- 当 `state == 0` 时，所有等待线程被唤醒。
#### ​**2. 等待队列**​
- `AQS` 维护一个双向链表（`CLH` 队列变种），用于管理调用 `await()` 后阻塞的线程。
- 当 `state` 未归零时，新调用 `await()` 的线程会被封装为节点并加入队列尾部，然后通过 `park()` 阻塞。

#### 三、关键方法解析
##### ​1. `countDown()`：递减计数器​
- ​**作用**​：将计数器减 1（若计数器已为 0，则无效果）。
- ​**实现**​：通过 `CAS` 原子操作更新 `state`（`compareAndSetState`）。若更新成功且 `state` 变为 0，则唤醒等待队列中的下一个线程。

##### ​2. `await()`：阻塞等待计数器归零​
- ​**作用**​：当前线程阻塞，直到计数器归零或被中断。
- ​**实现**​：
    1. 检查 `state` 是否为 0（若是，直接返回）。
    2. 若不为 0，将当前线程封装为节点并加入等待队列。
    3. 循环检查前驱节点是否为头节点（或是否被唤醒），若未满足条件则调用 `park()` 阻塞线程。

##### ​3. `await(long timeout, TimeUnit unit)`：带超时的等待​
- ​**作用**​：等待指定时间后自动唤醒（无论计数器是否归零）。
- ​**实现**​：类似 `await()`，但增加超时判断。若超时后 `state` 仍未归零，线程被唤醒并返回 `false`；若归零则返回 `true`。

#### 四、典型使用场景
##### 1. 主线程等待所有子线程完成任务​
例如：主线程启动多个子线程执行任务（如数据加载、文件下载），需等待所有子线程完成后汇总结果。
```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        int taskCount = 3;
        CountDownLatch latch = new CountDownLatch(taskCount);

        for (int i = 0; i < taskCount; i++) {
            new Thread(() -> {
                try {
                    // 模拟任务执行
                    Thread.sleep(1000);
                    System.out.println(Thread.currentThread().getName() + " 任务完成");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    latch.countDown(); // 任务完成后递减计数器
                }
            }).start();
        }

        latch.await(); // 主线程等待所有任务完成
        System.out.println("所有任务完成，主线程继续执行");
    }
}
```

##### ​**2. 多线程协作完成初始化**​
例如：多个模块初始化完成后，主线程启动系统。每个模块初始化完成后调用 `countDown()`，主线程通过 `await()` 等待所有模块就绪。

##### ​**3. 替代 `Object.wait/notify`**​
相比 `wait/notify`，`CountDownLatch` 更简洁，无需手动管理锁和条件变量，适用于多线程等待固定数量任务的场景。

#### 五、常见问题与注意事项
##### 1. `CountDownLatch` 与 `CyclicBarrier` 的区别
| ​**特性**​    | ​**CountDownLatch**​   | ​**CyclicBarrier**​ |
| ----------- | ---------------------- | ------------------- |
| ​**计数器方向**​ | 递减（从 `count` → 0）      | 递增（从 0 → `parties`） |
| ​**重用性**​   | 一次性（归零后无法重置）           | 可重复（到达屏障后可重置继续使用）   |
| ​**等待条件**​  | 等待计数器归零                | 等待线程数达到 `parties`   |
| ​**触发条件**​  | 最后一个线程调用 `countDown()` | 最后一个线程调用 `await()`  |
##### 2. 计数器必须递减到 0 吗？​​
是的。若计数器未归零，调用 `await()` 的线程会一直阻塞（除非超时或被中断）。因此，需确保所有任务线程最终都会调用 `countDown()`（即使任务异常，也应在 `finally` 块中调用）。
##### ​3. 超时等待的意义​
`await(timeout, unit)` 避免主线程无限等待（如子线程因死锁或错误无法完成任务），超时后可执行降级逻辑（如重试或记录日志）。
##### ​4. 中断处理​
若等待线程被中断（调用 `interrupt()`），`await()` 会抛出 `InterruptedException`，需在代码中处理（如清理资源或重新尝试）。

#### 六、常见面试题
1. ​`CountDownLatch` 的核心作用是什么？​**​  
    协调多个线程，允许一个或多个线程等待其他线程完成操作后再继续执行（通过计数器实现）。
    
2. ​`CountDownLatch` 如何保证线程安全？​**​  
    底层通过 `AQS` 的 `CAS` 操作（`compareAndSetState`）原子性地更新计数器（`state`），确保多线程下的可见性和原子性。
    
3. ​`CountDownLatch` 是一次性的吗？如何重复使用？​**​  
    是。计数器归零后无法重置。若需重复使用，可结合 `ReentrantLock` 和 `Condition` 自定义实现，或在任务完成后重新创建实例。
    
4. ​`await()` 方法的返回值有什么意义？​**​  
    无返回值（`void`），但带超时的 `await(long, TimeUnit)` 返回 `boolean`（`true` 表示计数器归零，`false` 表示超时）。
    
5. ​`CountDownLatch` 和 `CyclicBarrier` 的主要区别是什么？​**​  
    `CountDownLatch` 计数器递减（一次性），`CyclicBarrier` 计数器递增（可重复）；前者等待任务完成，后者等待线程数量。
    
6. ​**如何确保所有子线程都调用 `countDown()`？​**​  
    在子线程的 `finally` 块中调用 `countDown()`，避免因异常导致计数器未递减。