---
创建时间: 2025-06-05 22:41:07
作者: wangxiaoming
tags:
  - CyclicBarrier
  - ReentrantLock
  - Condition
---

Java 中的 ​`CyclicBarrier`**​ 是 `java.util.concurrent` 包下的同步辅助类，用于协调多个线程的执行顺序，允许一组线程在某个“屏障点”等待，直到所有线程都到达该点后再继续执行后续操作。其核心特点是**可重复使用**和**计数器递增**，适用于多阶段任务协作。以下是其核心考点的详细梳理：
#### 一、核心功能与设计思想
`CyclicBarrier` 的核心是一个**可递增的计数器**，初始化时设定一个正整数值（`parties`，表示需要等待的线程数）。每个线程完成任务后调用 `await()` 方法通知屏障；当所有 `parties` 个线程都调用了 `await()`，屏障会被“打开”，所有等待线程被唤醒并继续执行；之后屏障自动重置（计数器归零并重新开始计数），可重复用于下一轮协作。
##### **关键特性**​：
- ​**可重复性**​：屏障打开后自动重置，支持多轮协作（区别于 `CountDownLatch` 的一次性）。
- ​**多线程同步**​：支持多个线程在屏障点同步，等待所有线程到达后统一放行。
- ​**可选回调**​：构造时可传入 `Runnable` 作为“屏障动作”（所有线程到达后，由最后一个到达的线程执行）。

#### 二、底层实现：基于`ReentrantLock`和`Condition`
`CyclicBarrier` 底层通过 ​`ReentrantLock`**​（可重入锁）和 ​`Condition`​（条件变量）实现线程的阻塞与唤醒，核心字段包括：
##### 1. 核心字段​
- `parties`：需要等待的线程总数（初始化时指定）。
- `count`：剩余需要等待的线程数（初始为 `parties`，每调用一次 `await()` 减 1）。
- `generation`：代际标识（用于区分不同轮次的屏障，重置时递增）。
- `lock`：`ReentrantLock` 实例，用于同步对内部状态的访问。
- `trip`：`Condition` 实例，用于阻塞等待线程，直到屏障打开。
##### ​2. 关键流程​
- ​**线程等待（`await()`）​**​：
    1. 获取 `lock` 锁。
    2. 检查当前代际的 `count` 是否为 0（若是，说明是新一轮屏障，重置 `count` 并返回）。
    3. 若 `count > 0`，`count` 减 1；若 `count == 0`（最后一个到达的线程），执行屏障动作（若有），并唤醒所有等待线程。
    4. 若 `count > 0`，当前线程通过 `trip.await()` 阻塞，直到被唤醒。
- ​**屏障重置**​：  
    当所有线程到达后，`count` 重置为 `parties`，`generation` 递增（标记为新的一轮），等待下一轮协作。

#### 三、关键方法解析
##### 1. `await()`：阻塞等待屏障打开​
- ​**作用**​：当前线程在屏障点阻塞，直到所有 `parties` 个线程都调用了 `await()`，或被中断/超时。
- ​**实现**​：
    - 若当前线程是最后一个到达的线程（`count == 0`），执行屏障动作（`Runnable`），并唤醒所有等待线程。
    - 否则，线程进入 `trip` 条件变量的等待队列，释放锁并阻塞。

##### ​2. `await(long timeout, TimeUnit unit)`：带超时的等待​
- ​**作用**​：等待指定时间后自动唤醒（无论是否所有线程到达）。
- ​**实现**​：类似 `await()`，但增加超时判断。若超时后仍有线程未到达，屏障被破坏，唤醒所有线程并抛出 `BrokenBarrierException`。

##### ​3. `reset()`：手动重置屏障​
- ​**作用**​：强制重置屏障，将 `count` 恢复为 `parties`，并唤醒所有等待线程（可能导致 `BrokenBarrierException`）。
#### 四、典型使用场景
##### 1. 多线程分阶段计算​
例如：多个线程分别计算不同数据块的结果，完成后在屏障点等待，所有线程完成后合并结果。
```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        int threadCount = 3;
        CyclicBarrier barrier = new CyclicBarrier(threadCount, () -> {
            System.out.println("所有线程完成计算，开始合并结果...");
        });

        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                try {
                    // 模拟计算
                    System.out.println(Thread.currentThread().getName() + " 计算完成");
                    barrier.await(); // 等待其他线程
                    // 合并结果（屏障打开后执行）
                    System.out.println(Thread.currentThread().getName() + " 合并结果");
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}
```
##### ​2. 多阶段任务协作​
例如：游戏服务器中，多个玩家准备完成后统一开始游戏；或分布式系统中，多个节点完成本地任务后统一同步。
##### ​3. 替代 `Object.wait/notify` 或 `CountDownLatch`​
相比 `wait/notify`，`CyclicBarrier` 更简洁，无需手动管理锁和条件变量；相比 `CountDownLatch`，支持多轮协作。

#### 五、常见问题与注意事项
##### 1. `CyclicBarrier` 与 `CountDownLatch` 的区别
| ​**特性**​    | ​**CyclicBarrier**​ | ​**CountDownLatch**​   |
| ----------- | ------------------- | ---------------------- |
| ​**计数器方向**​ | 递增（从 `parties` → 0） | 递减（从 `count` → 0）      |
| ​**重用性**​   | 可重复（自动重置）           | 一次性（归零后无法重置）           |
| ​**等待条件**​  | 等待线程数达到 `parties`   | 等待计数器归零                |
| ​**触发条件**​  | 最后一个线程调用 `await()`  | 最后一个线程调用 `countDown()` |

##### 2. 屏障动作（`Runnable`）的执行时机​
屏障动作由最后一个到达屏障的线程执行（保证原子性），通常用于汇总或触发下一阶段操作（如合并计算结果）。
##### ​3. 异常处理​
- ​`InterruptedException`**​：等待线程被中断，屏障被破坏，其他等待线程会抛出 `BrokenBarrierException`。
- ​`BrokenBarrierException`**​：屏障被破坏（如超时、中断或重置），所有等待线程被唤醒并抛出此异常。
##### ​4. 重置（`reset()`）的风险​
手动调用 `reset()` 会唤醒所有等待线程（可能抛出 `BrokenBarrierException`），需谨慎使用（通常用于强制终止当前轮次协作）。
##### ​5. 可重入性​
同一线程可多次调用 `await()`（需在屏障打开后重新等待），但需确保逻辑正确（避免死锁）。

#### 六、常见面试题
1. **`CyclicBarrier` 的核心作用是什么？​**​  
    协调多个线程在屏障点同步，直到所有线程到达后统一放行，支持多轮协作。
    
2. ​`CyclicBarrier` 如何实现可重复使用？​**​  
    通过 `generation` 代际标识区分不同轮次，屏障打开后重置 `count` 并递增 `generation`，等待下一轮协作。
    
3. ​**屏障动作（`Runnable`）的作用是什么？由哪个线程执行？​**​  
    屏障动作是所有线程到达后的回调（如汇总结果），由最后一个调用 `await()` 的线程执行。
    
4. ​`CyclicBarrier` 和 `CountDownLatch` 的主要区别是什么？​**​  
    `CyclicBarrier` 计数器递增（可重复），`CountDownLatch` 计数器递减（一次性）；前者等待线程数，后者等待任务完成。
    
5. ​`await(timeout, unit)` 的超时机制如何工作？​**​  
    若超时后仍有线程未到达屏障，屏障被破坏，唤醒所有线程并抛出 `BrokenBarrierException`。
    
6. ​**如何避免 `CyclicBarrier` 的屏障破坏？​**​  
    确保所有线程最终都会调用 `await()`（如在 `finally` 块中调用），或在超时后手动处理异常。