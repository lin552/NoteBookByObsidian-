---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - 多线程
  - 业务经验
---

#### 一、单一线程池（`newSingleThreadExecutor`）
原理：通过 `Executors.newSingleThreadExecutor()` 创建仅含一个线程的线程池，所有提交的任务按 ​**FIFO（先进先出）​** 顺序执行

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(() -> System.out.println("Task 1"));
executor.submit(() -> System.out.println("Task 2"));
executor.shutdown();
```
**适用场景**：  
需要严格按提交顺序执行任务的场景（如日志记录、异步任务队列）。

#### 二、同步工具类

##### 1）`CountDownLatch`
原理：通过计数器实现线程等待。主线程等待子线程完成指定任务后继续执行

```java
CountDownLatch latch1 = new CountDownLatch(1);
CountDownLatch latch2 = new CountDownLatch(1);

Thread t1 = () -> { /* 任务 */ latch1.countDown(); };
Thread t2 = () -> { latch1.await(); /* 任务 */ latch2.countDown(); };
Thread t3 = () -> { latch2.await(); /* 任务 */ };

t1.start(); t2.start(); t3.start();
```
**适用场景**：  
多阶段任务依赖（如 T2 需等 T1 完成初始化）。

##### 2）`CyclicBarrier`
原理：通过屏障点（Barrier）让多个线程互相等待，达到屏障后同时继续执行

```java
CyclicBarrier barrier = new CyclicBarrier(3);
Thread t1 = () -> { barrier.await(); /* 任务 */ };
Thread t2 = () -> { barrier.await(); /* 任务 */ };
```

**适用场景**：  
并行计算中多个子任务需同步进度。

##### 3）Semaphore
原理：通过信号量控制并发许可数，间接实现顺序控制

```java
Semaphore semaphore = new Semaphore(0);
Thread t1 = () -> { /* 任务 */ semaphore.release(); };
Thread t2 = () -> { semaphore.acquire(); /* 任务 */ };
```

**适用场景**：  
资源池化（如数据库连接池）或限流场景。

##### 4)锁机制 （synchronized / Lock + Condition）

###### synchronized 
原理：通过对象锁强制线程互斥访问资源，配合 `wait()`/`notify()` 实现顺序控制

```java
Object lock = new Object();
Thread t1 = () -> { synchronized(lock) { /* 任务 */ lock.notify(); } };
Thread t2 = () -> { synchronized(lock) { lock.wait(); /* 任务 */ } };
```

**适用场景**：  
简单互斥操作（如单例模式）。

###### ReentrantLock + Condition
原理：通过 `Condition` 实现精确的线程唤醒机制

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();

Thread t1 = () -> { lock.lock(); /* 任务 */ condition.signal(); lock.unlock(); };
Thread t2 = () -> { lock.lock(); condition.await(); /* 任务 */ lock.unlock(); };
```

**适用场景**：  
复杂条件判断的同步（如生产者-消费者模型）。

#### 三、异步编排 （CompletableFuture）
原理：通过链式调用 `thenApply()`、`thenRun()` 等方法定义任务依赖关系

```java
CompletableFuture.runAsync(() -> System.out.println("Task 1"))
    .thenRun(() -> System.out.println("Task 2"))
    .thenRun(() -> System.out.println("Task 3"));
```

**适用场景**：  
异步任务流水线（如微服务调用链）。
#### 四、线程优先级 （Thread.setPriority）
原理：通过设置线程优先级（1~10）提示调度器优先执行，但 ​**不保证严格顺序**​（依赖操作系统实现）

```java
Thread t1 = new Thread(() -> {}, "HighPriorityThread");
t1.setPriority(Thread.MAX_PRIORITY);
```

**适用场景**：  
辅助性优化，不推荐作为核心顺序控制手段

#### 五、总结

| **方法**                    | ​**优点**    | ​**缺点**  | ​**适用场景** |
| ------------------------- | ---------- | -------- | --------- |
| 单一线程池                     | 简单、天然顺序性   | 并发性能低    | 串行任务队列    |
| `CountDownLatch`          | 灵活控制多阶段依赖  | 计数器一次性使用 | 分阶段任务协同   |
| `ReentrantLock+Condition` | 精确唤醒、支持公平锁 | 需手动管理锁释放 | 复杂条件同步    |
| `CompletableFuture`       | 异步编排、代码简洁  | 需理解函数式编程 | 异步任务流水线   |
| 线程优先级                     | 轻量级提示      | 不可靠、平台依赖 | 辅助性优化     |
