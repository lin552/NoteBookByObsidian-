---
创建时间: 2025-04-12 17:34:32
作者: wangxiaoming
tags:
  - 多线程
  - Java
  - 线程
---
Java 中的 `Thread` 类是多线程编程的核心工具，用于创建和管理线程。其考点覆盖**线程创建与生命周期**、**状态转换**、**同步机制**、**常用方法**及**线程池**等核心内容。以下是详细梳理：

#### 一、线程的创建与启动
##### 1. 创建线程的四种方式​
- ​**方式 1：继承 `Thread` 类**​  
    重写 `run()` 方法，通过 `start()` 启动线程。
    ```java
    class MyThread extends Thread {
        @Override
        public void run() {
            // 线程执行逻辑
        }
    }
    MyThread thread = new MyThread();
    thread.start(); // 启动线程（调用 run()）
    ```
    - ​**缺点**​：Java 单继承，限制扩展性。
    
- ​**方式 2：实现 `Runnable` 接口**​  
    实现 `run()` 方法，通过 `Thread` 包装后启动。
    ```java
    class MyRunnable implements Runnable {
        @Override
        public void run() {
            // 线程执行逻辑
        }
    }
    Thread thread = new Thread(new MyRunnable());
    thread.start();
    ```
    - ​**优点**​：解耦线程任务与线程本身，支持多实现。
    
- ​**方式 3：实现 `Callable` 接口（结合 `FutureTask`）​**​  
    实现 `call()` 方法（可返回结果或抛出异常），通过 `FutureTask` 包装后启动。
    ```java
    class MyCallable implements Callable<String> {
        @Override
        public String call() throws Exception {
            return "任务结果";
        }
    }
    FutureTask<String> futureTask = new FutureTask<>(new MyCallable());
    Thread thread = new Thread(futureTask);
    thread.start();
    String result = futureTask.get(); // 阻塞获取结果
    ```

    - ​**优点**​：支持返回值和异常捕获，适合需要结果的异步任务。
    
- ​**方式 4：线程池（`ExecutorService`）​**​  
    通过 `Executors` 工厂类或 `ThreadPoolExecutor` 创建线程池，提交任务（`Runnable`/`Callable`）。
    ```java
    ExecutorService executor = Executors.newFixedThreadPool(5);
    executor.submit(() -> { // 提交 Runnable 任务
        // 线程执行逻辑
    });
    executor.submit(() -> "任务结果"); // 提交 Callable 任务（返回 Future）
    ```
    - ​**优点**​：复用线程，避免频繁创建/销毁线程的开销。

#### 二、线程的生命周期与状态转换
Java 线程的状态由 `Thread.State` 枚举定义，共 6 种状态，转换关系如下：

| ​**状态**​              | ​**描述**​                                                             | ​**转换条件**​                                                                                                                                     |
| --------------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `NEW`（新建）             | 线程对象已创建，但未调用 `start()`。                                              | 调用 `start()` → `RUNNABLE`。                                                                                                                     |
| `RUNNABLE`（可运行）       | 线程已启动，正在 `JVM` 中运行（可能等待 CPU 调度）或在操作系统中运行。                            | - 获得 CPU 时间片 → 运行；  <br>- 等待锁/资源 → `BLOCKED`；  <br>- 调用 `wait()`/`join()` → `WAITING`；  <br>- 调用 `sleep(time)`/`join(time)` → `TIMED_WAITING`。 |
| `BLOCKED`（阻塞）         | 线程因等待获取对象锁（`synchronized`）而被阻塞。                                      | 获取到锁 → `RUNNABLE`。                                                                                                                             |
| `WAITING`（等待）         | 线程因调用 `wait()`、`join()`（无超时）或 `LockSupport.park()` 进入等待。             | 其他线程调用 `notify()`/`notifyAll()`（`wait()` 场景）或被 `join()` 的线程终止（`join()` 场景）→ `RUNNABLE`。                                                        |
| `TIMED_WAITING`（超时等待） | 线程因调用 `sleep(time)`、`join(time)` 或 `LockSupport.parkNanos()` 进入超时等待。 | 超时时间到 → `RUNNABLE`；  <br>- 被中断（`interrupt()`）→ 抛出 `InterruptedException`。                                                                      |
| `TERMINATED`（终止）      | 线程执行完 `run()` 或因异常终止。                                                | 无后续状态转换。                                                                                                                                       |
#### 三、线程的核心方法
##### ​1. `start()` vs `run()`​
- `start()`：启动线程，`JVM` 调用 `run()` 方法，线程进入 `RUNNABLE` 状态（可能等待 CPU 调度）。
- `run()`：线程的执行逻辑，直接调用 `run()` 不会启动新线程（等同于普通方法调用）。

##### ​2. `join()`：等待线程终止​
- ​**作用**​：当前线程阻塞，直到目标线程终止（或超时）。
- ​**示例**​：
    ```java
    Thread thread = new Thread(() -> {
        // 任务逻辑
    });
    thread.start();
    thread.join(); // 主线程等待 thread 终止后继续执行
    ```
    
##### ​3. `sleep(long millis)`：休眠线程
- ​**作用**​：让当前线程暂停执行 `millis` 毫秒（不释放锁）。
- ​**特点**​：
    - 进入 `TIMED_WAITING` 状态。
    - 可能抛出 `InterruptedException`（被中断时）。

##### ​4. `yield()`：让步 CPU​
- ​**作用**​：提示 `JVM` 当前线程愿意让出 CPU 时间片（但不保证立即切换）。
- ​**特点**​：
    - 线程仍处于 `RUNNABLE` 状态。
    - 仅用于优化线程调度（如避免某线程长时间占用 CPU）。

##### ​5. `interrupt()`：中断线程​
- ​**作用**​：设置线程的中断标志（非强制终止），需线程配合处理。
- ​**处理方式**​：
    - 检查中断标志（`Thread.currentThread().isInterrupted()`）。
    - 响应中断（如退出循环、抛出 `InterruptedException`）。

    ```java
    Thread thread = new Thread(() -> {
        while (!Thread.currentThread().isInterrupted()) {
            // 任务逻辑
        }
        System.out.println("线程被中断");
    });
    thread.start();
    thread.interrupt(); // 设置中断标志
    ```

##### ​6. `stop()`：强制终止线程（已过时）​​
- ​**作用**​：强制终止线程（不推荐使用，可能导致资源未释放、数据不一致）。
- ​**替代方案**​：通过标志位或 `interrupt()` 优雅终止线程。

#### 四、线程同步与锁机制
##### 1. `synchronized` 关键字​
- ​**作用**​：保证多线程下共享资源的互斥访问（原子性、可见性、有序性）。
- ​**使用方式**​：
    - ​**对象锁**​：同步代码块使用 `synchronized(obj)`。
    - ​**类锁**​：同步代码块使用 `synchronized(ClassName.class)` 或静态方法。
    - ​**方法锁**​：同步方法（锁是当前对象或类）。
- ​**底层实现**​：依赖`JVM` 的 `Monitor` 锁（对象头的 `Mark Word` 存储锁状态）。

##### ​2. `Lock` 接口（`ReentrantLock`）​​
- ​**作用**​：比 `synchronized` 更灵活的锁机制（可中断、超时获取、公平锁）。
- ​**核心方法**​：
    - `lock()`：获取锁（阻塞）。
    - `tryLock()`：尝试获取锁（非阻塞）。
    - `lockInterruptibly()`：可中断获取锁。
    - `unlock()`：释放锁（需手动调用）。
- ​**示例**​：
    ```java
    Lock lock = new ReentrantLock();
    lock.lock();
    try {
        // 临界区
    } finally {
        lock.unlock(); // 确保释放锁
    }
    ```

##### ​3. `volatile` 关键字​
- ​**作用**​：保证变量的**可见性**​（修改后立即刷新到主内存）和**有序性**​（禁止指令重排）。
- ​**适用场景**​：
    - 状态标志（如 `volatile boolean running`）。
    - 单例模式的双重检查锁定（`DCL`）。

##### ​4. 死锁与解决​
- ​**死锁条件**​：互斥、占有并等待、不可抢占、循环等待。
- ​**检测与解决**​：
    - 通过 `jstack` 工具查看线程堆栈，定位死锁。
    - 避免嵌套锁、按顺序获取锁、使用超时锁（`tryLock`）。

#### 五、线程池(`ExecutorService`)
##### 1. 核心组件​
- ​**`ThreadPoolExecutor`**​：线程池的核心实现类，关键参数：
    - `corePoolSize`：核心线程数（常驻线程）。
    - `maxPoolSize`：最大线程数（临时线程上限）。
    - `keepAliveTime`：非核心线程空闲存活时间。
    - `workQueue`：任务队列（如 `LinkedBlockingQueue`、`ArrayBlockingQueue`）。
    - `threadFactory`：线程工厂（自定义线程名称、优先级）。
    - `handler`：拒绝策略（任务队列满时的处理方式，如 `AbortPolicy`、`CallerRunsPolicy`）。

##### ​2. 常见线程池类型​
- ​**`newFixedThreadPool`**​：固定核心线程数（`maxPoolSize = corePoolSize`），任务队列为无界队列（可能导致`OOM`）。
- ​**`newCachedThreadPool`**​：核心线程数 0，最大线程数无界，任务队列为同步移交队列（适合短任务）。
- ​**`newSingleThreadExecutor`**​：单核心线程（顺序执行任务，避免并发问题）。
- ​**`newScheduledThreadPool`**​：支持定时/周期性任务（如 `scheduleAtFixedRate`）。

#### 六、常见问题与面试题
1. ​**线程的生命周期包括哪些状态？如何转换？​**​  
    新建（`NEW`）→ 可运行（`RUNNABLE`）→ 阻塞（`BLOCKED`）/等待（`WAITING`）/超时等待（`TIMED_WAITING`）→ 终止（`TERMINATED`）。具体转换条件需结合 `synchronized`、`wait()`、`sleep()` 等操作。
    
2. ​`start()` 和 `run()` 的区别是什么？​**​  
    `start()` 启动新线程并调用 `run()`；`run()` 是线程执行逻辑，直接调用不会创建新线程。
    
3. ​`sleep()` 和 `wait()` 的区别是什么？​**​  
    `sleep()` 是 `Thread` 方法，不释放锁，进入 `TIMED_WAITING`；`wait()` 是 `Object` 方法，释放锁，进入 `WAITING`。
    
4. ​**如何优雅终止线程？​**​  
    通过 `interrupt()` 设置中断标志，线程内部检查标志并退出循环；或使用 `volatile` 标志位控制。
    
5. ​**死锁的原因和解决方法是什么？​**​  
    原因：互斥、占有并等待、不可抢占、循环等待。解决方法：避免嵌套锁、按顺序获取锁、使用超时锁。
    
6. ​**线程池的作用是什么？如何合理配置？​**​  
    作用：复用线程、控制并发数、避免资源耗尽。配置需根据任务类型（CPU 密集型/IO 密集型）调整 `corePoolSize` 和 `maxPoolSize`。