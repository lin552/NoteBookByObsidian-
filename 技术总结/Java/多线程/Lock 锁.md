---
创建时间: 2025-03-14T18:39:00
作者: wangxiaoming
tags:
  - 多线程
  - Lock
---

Java 中的 `Lock` 接口是 `java.util.concurrent.locks` 包的核心组件，提供了比 `synchronized` 更灵活、更强大的线程同步机制。
#### 一、Lock的引入背景与核心优势
##### 1）解决 `synchronized` 的局限性
- ​**不可中断性**：`synchronized` 阻塞的线程无法响应中断请求，而 `Lock` 支持 `lockInterruptibly()` 方法实现可中断锁获取。
- ​**超时获取**：`Lock` 提供 `tryLock(timeout)` 方法，避免线程无限等待。
- ​**公平性**：支持公平锁（按请求顺序分配锁）和非公平锁（默认），而 `synchronized` 仅支持非公平锁。
- ​**多条件变量**：通过 `Condition` 实现复杂线程通信（类似 `wait()/notify()` 但更灵活）。

##### 2）核心优势
- ​**显式锁管理**：需手动调用 `lock()` 和 `unlock()`，避免锁泄漏问题
- ​**细粒度控制**：适用于高并发场景，减少锁竞争开销，提升吞吐量

#### 二、Lock的核心实现类与使用
##### 1）[[ReetrantLock 可重入锁]] 
```java
ReentrantLock lock = new ReentrantLock();  
lock.lock();  
try {  
    // 临界区代码  
} finally {  
    lock.unlock();  // 必须显式释放锁[1,6](@ref)  
}  
```
**高级特性**：
- ​**公平锁**：通过 `new ReentrantLock(true)` 创建，按线程请求顺序分配锁
- ​**锁状态查询**：`getHoldCount()` 获取当前线程持有锁的次数，`isLocked()` 判断锁是否被占用
##### 2）[[ReentrantReadWriteLock 读写锁]]
**分离读写锁**：允许多个读线程同时访问，写线程独占资源
```java
ReadWriteLock lock = new ReentrantReadWriteLock();  
Lock readLock = lock.readLock();  
Lock writeLock = lock.writeLock();  
```
**适用场景**：读多写少的缓存系统，如配置管理

#### 三、Lock的底层实现：[[AQS (AbstractQueuedSynchronizer)]]
1. ​**核心机制**
    - ​**同步状态管理**：通过 `int` 类型变量表示锁状态（如 `ReentrantLock` 记录重入次数）。
    - ​**线程队列**：内置 FIFO 队列管理等待线程，支持独占锁和共享锁模式。
    - ​**CAS 操作**：通过 `compareAndSetState()` 实现无锁化状态更新，确保原子性。
2. ​**锁的扩展性**
    - ​**自定义锁**：通过继承 AQS 并实现 `tryAcquire()`/`tryRelease()` 方法，可快速构建定制化锁

#### 四、Lock与synchronized的对比
|​**维度**|`Lock`|`synchronized`|
|---|---|---|
|​**锁获取方式**|显式调用 `lock()`/`unlock()`|隐式（代码块或方法）|
|​**公平性**|支持公平和非公平锁|仅非公平锁|
|​**可中断性**|支持（`lockInterruptibly()`）|不支持|
|​**超时机制**|支持（`tryLock(timeout)`）|不支持|
|​**条件变量**|支持多个 `Condition`|单一 `wait()/notify()`|
|​**性能**|高并发下更高效|低竞争时性能较好（JVM 优化）|
#### 五、典型使用场景与最佳实践
##### 1）场景实例
###### 生产者-消费者模型：通过Condition实现精确线程通信
```java
ReentrantLock lock = new ReentrantLock();  
Condition notFull = lock.newCondition();  
Condition notEmpty = lock.newCondition();  
// 生产者：notFull.await() 和 notEmpty.signalAll()  
// 消费者：notEmpty.await() 和 notFull.signalAll()  
```
**高并发计数器**：利用 `ReentrantLock` 保证原子性

##### 2）最佳实践
- ​**锁释放保障**：`unlock()` 必须放在 `finally` 块中，避免死锁
- ​**减少锁粒度**：仅锁定必要代码段，提升并发性能
- ​**避免锁嵌套**：防止死锁，按固定顺序获取多把锁。

