---
创建时间: 2025-04-12 17:34:32
作者: wangxiaoming
tags:
  - Java
  - 多线程
  - 原子类
  - CAS
---

Java 中的 `​CAS`（Compare-And-Swap，比较并交换）​**​ 是一种无锁的原子操作机制，广泛用于实现线程安全的共享变量修改。其核心思想是“乐观锁”：假设多线程竞争不激烈，通过“比较-更新”的循环尝试完成原子操作，避免传统锁的阻塞开销。以下是其核心考点的详细梳理：
#### 一、`CAS`的核心原理
`CAS` 是一种基于硬件支持的原子操作指令，依赖 CPU 提供的**原子指令**​（如 `x86` 的 `CMPXCHG`）实现。其底层逻辑可概括为三个步骤：
1. ​**读取**​：获取共享变量的当前值（记为 `V`）。
2. ​**比较**​：检查 `V` 是否等于预期的旧值（记为 `A`）。
3. ​**交换**​：若相等（`V == A`），则将变量更新为新值（记为 `B`）；否则不操作。
##### ​**1. Java 中的 `CAS` 实现**​
Java 中通过 `sun.misc.Unsafe` 类（或 `JDK 9+` 的 `VarHandles`）调用本地方法实现 `CAS`。例如，`AtomicInteger` 的 `incrementAndGet()` 方法底层即通过 `CAS` 完成无锁递增：
```java
public final int incrementAndGet() {
    return U.getAndAddInt(this, VALUE, 1) + 1;
}
// Unsafe 的 getAndAddInt 方法伪代码（依赖 CPU 原子指令）
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset); // 读取当前值 V
    } while (!compareAndSwapInt(o, offset, v, v + delta)); // CAS 尝试更新
    return v;
}
```
#### 二、`CAS`的典型应用
`CAS` 是许多并发工具类的底层基础，常见场景包括：
##### **1. 原子类（`AtomicXXX`）​**​
`java.util.concurrent.atomic` 包下的类（如 `AtomicInteger`、`AtomicLong`、`AtomicReference`）均基于 `CAS` 实现无锁原子操作。例如：
- `AtomicInteger` 的 `addAndGet(int delta)`：通过 `CAS` 循环实现“读取-修改-写入”的原子性。
- `AtomicReference`：支持对象引用的原子更新（避免多线程下的可见性问题）。
##### ​**2. 非阻塞数据结构**​
如无锁队列（`ConcurrentLinkedQueue`）、无锁栈等，通过 `CAS` 实现节点的插入、删除操作，避免锁的阻塞。
#### ​**3. 并发工具类**​
- `ReentrantLock` 的 `tryLock()` 方法：尝试通过 `CAS`获取锁状态（非公平锁）。
- `CountDownLatch`、`CyclicBarrier` 等同步工具的内部计数器更新。
- `ConcurrentHashMap`（`JDK 8+`）：在扩容、节点插入时使用 `CAS` 减少锁竞争。
#### 三、`CAS`的优缺点
##### ​**1. 优点**​
- ​**无锁，低开销**​：避免了传统锁（如 `synchronized`）的线程阻塞和上下文切换，适用于多线程竞争不激烈的场景。
- ​**高并发性能**​：通过循环重试（自旋）完成操作，减少线程等待时间。
##### ​**2. 缺点**​
- ​**ABA 问题**​：变量从 `A` → `B` → `A`，`CAS` 操作认为未变化，但中间可能有其他操作干扰（需额外处理）。
- ​**自旋开销**​：若竞争激烈，`CAS` 失败次数过多会导致自旋循环消耗大量 CPU 资源。
- ​**仅保证单变量原子性**​：无法直接保证多变量或多步操作的原子性（需结合其他机制）。

#### 四、`CAS`的核心问题与解决方案
##### ​**1. ABA 问题**​
- ​**定义**​：线程 1 读取变量值为 `A`，准备更新为 `B`；此时线程 2 将变量从 `A` 改为 `B`，再改回 `A`；线程 1 的 `CAS` 操作因 `V == A` 成功，但实际变量已被修改过。
- ​**影响**​：可能导致逻辑错误（如“对象已失效但仍被使用”）。
- ​**解决方案**​：
    - ​**版本号机制**​：使用 `AtomicStampedReference`（记录变量的版本号），`CAS` 时同时比较值和版本号。
    - ​**时间戳机制**​：类似版本号，通过时间戳标记变量的修改次数。
    ```java
    // AtomicStampedReference 示例（解决 ABA 问题）
    AtomicStampedReference<Integer> ref = new AtomicStampedReference<>(100, 0); // 初始值 100，版本号 0
    
    // 线程 1：尝试将 100 → 101（版本号 0 → 1）
    int oldValue = 100;
    int newVersion = ref.getStamp() + 1;
    boolean success = ref.compareAndSet(oldValue, 101, ref.getStamp(), newVersion);
    
    // 线程 2：先将 100 → 101（版本号 0 → 1），再将 101 → 100（版本号 1 → 2）
    ref.compareAndSet(100, 101, 0, 1); // 成功
    ref.compareAndSet(101, 100, 1, 2); // 成功
    
    // 线程 1 再次尝试 CAS（此时值回到 100，但版本号已变为 2）
    success = ref.compareAndSet(100, 200, 0, 3); // 失败（版本号不匹配）
    ```
##### ​**2. 自旋开销问题**​
- ​**定义**​：`CAS` 失败时，线程会循环重试（自旋），若竞争激烈，自旋次数过多会导致 CPU 资源浪费。
- ​**优化方案**​：
    - ​**限制自旋次数**​：超过阈值后放弃自旋，转为阻塞（如 `ReentrantLock` 的可重入锁实现）。
    - ​**自适应自旋**​：根据历史自旋成功率动态调整自旋次数（`JVM` 优化策略）。
#### 五、`CAS`与其他同步机制的对比
|​**特性**​|​**CAS（乐观锁）​**​|​**synchronized（悲观锁）​**​|
|---|---|---|
|锁类型|无锁（基于 CPU 原子指令）|重量级锁（内核互斥量）|
|竞争场景|低竞争（自旋次数少）|高竞争（阻塞等待）|
|上下文切换|无（用户态操作）|有（内核态切换）|
|适用场景|单变量原子操作、轻量级同步|复杂同步（互斥、条件等待）|
|实现复杂度|高（需处理 ABA、自旋等问题）|低（JVM 封装）|
#### 六、常见面试题
1. ​**`CAS` 的核心原理是什么？​**​  
    `CAS` 是一种无锁原子操作，通过比较变量的当前值与预期值，若相等则更新为新值（依赖 CPU 原子指令）。
    
2. ​`CAS` 如何保证原子性？​**​  
    依赖 CPU 提供的原子指令（如 `x86` 的 `CMPXCHG`），确保“比较-交换”操作的不可分割性。
    
3. ​**什么是 ABA 问题？如何解决？​**​  
    ABA 问题是变量从 `A` → `B` → `A` 时，`CAS` 误判为未变化。解决方案是使用带版本号的 `AtomicStampedReference` 或时间戳机制。
    
4. ​`CAS` 的缺点有哪些？​**​  
    自旋开销大（竞争激烈时 CPU 浪费）、仅保证单变量原子性、需处理 ABA 问题。
    
5. ​`CAS` 和 synchronized 的区别是什么？​**​  
    `CAS` 是无锁乐观锁（用户态操作），适用于低竞争场景；`synchronized` 是悲观锁（内核互斥），适用于高竞争场景。
    
6. ​**Java 中哪些类使用了 `CAS`？​**​  
    `AtomicInteger`、`AtomicReference`、`ReentrantLock`、`ConcurrentHashMap`（`JDK 8+`）等。