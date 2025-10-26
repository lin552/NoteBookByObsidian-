---
创建时间: 2025-06-06 00:09:48
作者: wangxiaoming
tags:
  - Java
  - 线程池
---
Java 线程池（`ThreadPoolExecutor`）是并发编程的核心工具，用于**复用线程、控制并发数、避免资源耗尽**，广泛应用于高并发场景（如Web服务器、批量任务处理）。以下是其核心考点的详细梳理：

#### 一、线程池的核心作用
- **复用线程**​：避免频繁创建/销毁线程的开销（创建线程需分配栈空间、`JVM`资源，销毁需`GC`回收）。
- ​**控制并发数**​：限制同时运行的线程数，防止资源耗尽（如CPU、内存）。
- ​**统一管理任务**​：提供任务队列、拒绝策略、线程工厂等机制，规范任务执行流程。

#### 二、线程池的核心参数（`ThreadPoolExecutor`构造函数）
线程池的行为由以下6个核心参数控制，理解它们是调优和解决问题的关键：

| ​**参数**​        | ​**类型**​                   | ​**描述**​                                                 |
| --------------- | -------------------------- | -------------------------------------------------------- |
| `corePoolSize`  | `int`                      | 核心线程数（常驻线程）。即使线程空闲，也不会销毁（除非设置 `allowCoreThreadTimeOut`）。 |
| `maxPoolSize`   | `int`                      | 最大线程数（临时线程上限）。当任务队列满且核心线程全忙时，可创建临时线程（不超过此值）。             |
| `keepAliveTime` | `long`                     | 非核心线程的空闲存活时间。超时后，临时线程会被销毁（减少资源占用）。                       |
| `unit`          | `TimeUnit`                 | `keepAliveTime` 的时间单位（如 `TimeUnit.SECONDS`）。             |
| `workQueue`     | `BlockingQueue<Runnable>`  | 任务队列。用于存放未执行的任务（核心线程满时，任务进入此队列等待）。                       |
| `threadFactory` | `ThreadFactory`            | 线程工厂（可选）。自定义线程名称、优先级、是否为守护线程等。                           |
| `handler`       | `RejectedExecutionHandler` | 拒绝策略（可选）。当任务队列满且线程数达到 `maxPoolSize` 时，对新任务的处理策略。         |
|                 |                            |                                                          |
#### 三、线程池的工作流程
任务提交后，线程池按以下流程处理任务（以 `execute(Runnable command)` 为例）：
1. ​**检查核心线程数**​：若当前运行的线程数 `< corePoolSize`，创建新线程（核心线程）执行任务。
2. ​**任务入队**​：若核心线程数已满，将任务加入 `workQueue` 等待。
3. ​**创建临时线程**​：若任务队列已满且当前线程数 `< maxPoolSize`，创建临时线程执行任务。
4. ​**执行拒绝策略**​：若任务队列已满且线程数 ≥ `maxPoolSize`，触发拒绝策略（如丢弃任务、抛出异常）。

#### 四、任务队列（`workQueue`）的类型与影响
任务队列是线程池的“缓冲区”，不同类型的队列决定了线程池的行为：
##### ​1. 无界队列（Unbounded Queue）​​
- ​**特点**​：容量无限（如 `LinkedBlockingQueue`），任务永远入队，不会触发临时线程创建。
- ​**适用场景**​：任务量稳定且较少，避免频繁创建线程（如后台日志处理）。
- ​**风险**​：任务堆积可能导致内存溢出（`OOM`）。
##### ​2. 有界队列（Bounded Queue）​​
- ​**特点**​：容量有限（如 `ArrayBlockingQueue`），任务队列满后触发临时线程创建。
- ​**适用场景**​：需控制任务堆积（如电商秒杀活动的订单处理）。
##### ​3. 同步移交队列（Synchronous Queue）​​
- ​**特点**​：不存储任务，直接将任务移交线程执行（若无线程可用则触发临时线程创建）。
- ​**适用场景**​：任务需立即执行（如高并发的短任务，如HTTP请求处理）。

#### 五、拒绝策略（`RejectedExecutionHandler`）
当任务队列满且线程数达到 `maxPoolSize` 时，线程池触发拒绝策略。Java 提供4种内置策略：

| ​**策略**​                  | ​**实现类**​                                | ​**行为**​                                  |
| ------------------------- | ---------------------------------------- | ----------------------------------------- |
| ​**AbortPolicy（默认）​**​    | `ThreadPoolExecutor.AbortPolicy`         | 抛出 `RejectedExecutionException` 异常，拒绝新任务。 |
| ​**CallerRunsPolicy**​    | `ThreadPoolExecutor.CallerRunsPolicy`    | 由调用者线程（提交任务的线程）直接执行任务（减缓任务提交速度）。          |
| ​**DiscardPolicy**​       | `ThreadPoolExecutor.DiscardPolicy`       | 静默丢弃新任务，不做任何处理。                           |
| ​**DiscardOldestPolicy**​ | `ThreadPoolExecutor.DiscardOldestPolicy` | 丢弃队列中最旧的任务，尝试重新提交当前任务（可能再次触发拒绝）。          |
#### 六、常见线程池类型（Executors工厂类）
Java 提供 `Executors` 工厂类快速创建预配置的线程池，但需注意其适用场景和潜在问题（如`OOM`）：

##### 1. `newFixedThreadPool(int n)`​
- ​**参数**​：核心线程数 = 最大线程数 = `n`，队列为无界 `LinkedBlockingQueue`。
- ​**特点**​：线程数固定，任务队列无界（可能`OOM`）。
- ​**适用场景**​：任务量稳定、并发数固定的场景（如数据库连接池）。

##### ​2. `newCachedThreadPool()`​
- ​**参数**​：核心线程数=0，最大线程数=无界（`Integer.MAX_VALUE`），队列为 `SynchronousQueue`。
- ​**特点**​：线程空闲60秒后销毁（节省资源），适合短任务（如HTTP请求）。
- ​**风险**​：高并发时可能创建大量线程（`Integer.MAX_VALUE` 约20亿），导致`OOM`。

##### ​3. `newSingleThreadExecutor()`​
- ​**参数**​：核心线程数=最大线程数=1，队列为无界 `LinkedBlockingQueue`。
- ​**特点**​：单线程顺序执行任务（避免并发问题），适合需要串行化的场景（如日志写入）。

##### ​4. `newScheduledThreadPool(int corePoolSize)`​
- ​**参数**​：核心线程数固定，支持定时/周期性任务（如 `scheduleAtFixedRate`）。
- ​**特点**​：线程空闲后存活时间为0（随用随创建），适合定时任务（如监控、报表生成）。

#### ​七、线程池的正确使用与调优

##### 1. 避免使用 `Executors` 直接创建**​
`Executors` 创建的线程池（如 `newFixedThreadPool`）使用无界队列，高并发时可能导致`OOM`。推荐直接使用 `ThreadPoolExecutor` 构造函数，明确配置参数。
##### ​2. 合理设置线程池大小​
- ​**CPU密集型任务**​：核心线程数 ≈ CPU核心数（避免线程切换开销）。
- ​**IO密集型任务**​：核心线程数 ≈ CPU核心数 × 2（IO等待时线程空闲，可增加线程数）。
##### ​3. 任务队列的选择​
- 短任务、高并发：使用 `SynchronousQueue`（直接移交线程）。
- 长任务、任务量稳定：使用有界队列（如 `ArrayBlockingQueue`）。
##### ​4. 拒绝策略的选择​
- 严格拒绝：`AbortPolicy`（默认，抛出异常）。
- 缓冲处理：`CallerRunsPolicy`（减缓提交速度）。
##### ​5. 线程池的关闭​
- `shutdown()`：优雅关闭（不再接受新任务，已提交任务继续执行）。
- `shutdownNow()`：强制关闭（尝试中断运行中的线程，返回未执行的任务列表）。

#### 八、高频面试题
1. ​**线程池的核心参数有哪些？各自的作用是什么？​**​  
    核心参数包括 `corePoolSize`、`maxPoolSize`、`keepAliveTime`、`workQueue`、`threadFactory`、`handler`，分别控制核心线程数、最大线程数、空闲存活时间、任务队列、线程工厂和拒绝策略。
    
2. ​**任务队列满且线程数达到 `maxPoolSize` 时，线程池会如何处理新任务？​**​  
    触发拒绝策略（默认抛出 `RejectedExecutionException`）。
    
3. ​**`newFixedThreadPool` 和 `newCachedThreadPool` 的区别是什么？​**​  
    `newFixedThreadPool` 线程数固定，使用无界队列；`newCachedThreadPool` 线程数动态调整（最大无界），使用同步移交队列，适合短任务。
    
4. ​**如何合理设置线程池的核心线程数和最大线程数？​**​  
    CPU密集型任务：核心线程数 ≈ CPU核心数；IO密集型任务：核心线程数 ≈ CPU核心数 × 2。
    
5. ​**线程池的 `shutdown()` 和 `shutdownNow()` 有什么区别？​**​  
    `shutdown()` 优雅关闭（不再接受新任务，已提交任务执行完毕）；`shutdownNow()` 强制关闭（中断运行中的线程，返回未执行任务）。
    
6. ​**为什么不建议使用 `Executors` 直接创建线程池？​**​  
    `Executors` 创建的线程池（如 `newFixedThreadPool`）使用无界队列，高并发时可能导致任务堆积和`OOM`。