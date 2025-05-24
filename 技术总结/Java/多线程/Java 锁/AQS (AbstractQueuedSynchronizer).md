---
创建时间: 2025-04-18 20:35:34
作者: wangxiaoming
tags:
  - AQS
  - ReetrantLock
  - Semaphore
  - CountDownLatch
---
#### 一、什么是`AQS`？
`AQS（AbstractQueuedSynchronizer）​​` 是 Java 并发包（`java.util.concurrent`）的核心框架，用于构建锁和同步器。它通过 ​**状态管理（state）​**​ 和 ​`CLH 队列`​（双向链表）实现线程的阻塞、排队与唤醒机制。几乎所有`JUC` 中的同步工具（如 `ReentrantLock`、`Semaphore`、`CountDownLatch`）均基于 `AQS` 实现
#### 二、核心原理
##### 1）状态变量（state）
- 通过 `volatile int state` 表示资源状态，子类可自定义语义。例如：
    - `ReentrantLock`：`state=0` 表示未加锁，`state>0` 表示重入次数
    - `Semaphore`：`state` 表示剩余许可证数量
- 通过 `compareAndSetState()` 原子操作修改状态
##### 2）`CLH`队列
- 采用 ​**虚拟双向队列**​ 管理等待线程，节点（`Node`）封装线程及状态（如 `SIGNAL` 表示需唤醒后继节点）
- ​**独占模式**​：仅一个线程可获取资源（如 `ReentrantLock`）；
- **共享模式**​：允许多线程共享（如 `Semaphore`）
##### 3）模板方法模式
1. - 子类需实现关键方法：
        - `tryAcquire()` / `tryRelease()`（独占模式）
        - `tryAcquireShared()` / `tryReleaseShared()`（共享模式）
#### 三、使用方式
##### 1）自定义锁示例
继承`AQS`并实现核心方法
```java
/**  
 * 基于AQS实现的不可重入的互斥锁  
 */  
public class MyMutex {  
  
    //内部同步器继承AQS  
    private static class Sync extends AbstractQueuedSynchronizer {  
        //尝试获取锁(独占模式)  
        @Override  
        protected boolean tryAcquire(int arg) {  
            //CAS操作:若state=0(未锁定)，则设置为1（锁定）  
            System.out.println("尝试获取锁 Thread " + Thread.currentThread().getId() + " <UNK>");  
            return compareAndSetState(0, 1);  
        }  
        //尝试释放锁  
        @Override  
        protected boolean tryRelease(int arg) {  
            setState(0); //直接释放锁，无需CAS(仅由持有线程调用)  
            System.out.println("尝试释放锁 Thread " + Thread.currentThread().getId() + " <UNK>");  
            return true;  
        }  
        //是否被当前线程独占  
        @Override  
        protected boolean isHeldExclusively() {  
            return getState() == 1;  
        }  
    }  
  
    private final Sync sync = new Sync();  
  
    //加锁  
    public void lock() {  
        sync.acquire(1);//调用AQS的模板方法  
    }  
  
    public void unlock() {  
        sync.release(1);  
    }  
  
}
```
##### 2）高级同步工具
- `ReentrantLock`：通过 `lock()`/`unlock()` 控制临界区
- `Condition`：结合 `await()`/`signal()` 实现条件等待（如生产者-消费者模型）
#### 四、技巧与注意事项
##### 1）技巧
- ​**避免锁泄漏**​：确保 `unlock()` 在 `finally` 块中调用
- ​**非公平锁优化**​：默认采用非公平模式（减少线程切换开销），但可能导致饥饿
- ​**条件变量优化**​：使用 `Condition` 替代 `Object.wait()`，支持多条件队列
##### 2）注意事项
- ​`CAS` 原子性：实现 `tryAcquire()` 时必须使用 `compareAndSetState()`，避免多线程竞争
- ​**中断处理**​：使用 `lockInterruptibly()` 替代 `lock()` 以支持响应中断
- ​**避免死锁**​：共享锁需注意释放顺序，防止循环等待
#### 五、适用场景
1. ​**高并发同步控制**​：如 `ReentrantLock` 替代 `synchronized`，提供更灵活的特性（如可中断、超时）
2. ​**资源池管理**​：如 `Semaphore` 控制数据库连接池并发数
3. ​**任务协调**​：如 `CountDownLatch` 实现多线程任务等待
4. ​**读写分离**​：如 `ReentrantReadWriteLock` 优化读多写少场景
#### 六、面试考察点
1. ​**核心原理**​
    - `AQS` 如何通过 state 和队列管理线程？
    - 独占模式与共享模式的区别？
    - `CLH` 队列的优化点（如双向链表设计）
2. ​**实现细节**​
    - `tryAcquire()` 的实现逻辑（如可重入锁的 state 累加）
    - 条件变量（`Condition`）如何与主队列交互？
3. ​**应用与设计**​
    - 如何基于 `AQS` 实现自定义同步器？
    - 公平锁与非公平锁的差异及适用场景
4. ​**性能与问题排查**​
    - `AQS` 如何通过 `CAS` 减少锁竞争？
    - 线程阻塞唤醒的底层实现（`LockSupport.park()`/`unpark()`）