---
创建时间: 2025-04-18 22:15:55
作者: wangxiaoming
tags:
  - ReetrantLock
---
#### 一、什么是`ReetrantLock`?
`ReentrantLock`​ 是 Java 并发包（`java.util.concurrent.locks`）中提供的 ​**可重入互斥锁**，是 `Lock` 接口的核心实现类。它通过 ​**显式加锁/释放锁**​ 的方式，提供了比 `synchronized` 更灵活的线程同步机制，支持 ​**可重入性、公平锁、可中断锁、超时锁**​ 等高级特性
#### 二、核心原理
##### 1）基于`AQS（AbstractQueuedSynchronized）` [[AQS (AbstractQueuedSynchronizer)]]
- ​`AQS` 框架**​：`ReentrantLock` 依赖 `AQS` 实现锁的排队、阻塞、唤醒逻辑。`AQS` 维护一个 ​**`CLH` 双向队列**​（同步队列）管理等待线程，并通过 `volatile int state` 表示锁状态
- ​**state 的语义**​：
    - `state=0`：锁未被占用。
    - `state>0`：锁被占用，数值表示当前线程的 ​**重入次数**​
- ​**公平性与非公平性**​：
    - ​**非公平锁（默认）​**​：新线程可插队竞争锁，性能更高（减少线程切换），但可能导致饥饿
    - ​**公平锁**​：按 `CLH` 队列顺序分配锁，避免饥饿，但性能较低
##### 2）可重入性实现
- 同一线程多次调用 `lock()` 时，`state` 递增；每次 `unlock()` 时递减，直到 `state=0` 才完全释放锁
- 通过 `exclusiveOwnerThread` 记录当前持有锁的线程，确保只有持有线程能重入

#### 三、使用方式
##### 1）基础用法
```java
/**  
 * ReentrantLock 基础使用  
 */  
public class ReentrantLockUse {  
  
    private final ReentrantLock lock = new ReentrantLock();  
    private int count = 0;  
  
    public void increment() {  
        lock.lock(); //加锁  
        try {  
            count++; //临界区代码  
        } finally {  
            lock.unlock(); //必须释放锁  
        }  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        ReentrantLockUse reentrantLockUse = new ReentrantLockUse();  
        Thread t1 = new Thread(()->{  
            for (int i = 0; i < 1000; i++) {  
                reentrantLockUse.increment();  
            }  
        });  
        Thread t2 = new Thread(()->{  
            for (int i = 0; i < 1000; i++) {  
                reentrantLockUse.increment();  
            }  
        });  
        t1.start();  
        t2.start();  
        t1.join();  
        t2.join();  
        System.out.println("Final count: " + reentrantLockUse.count);  
    }  
  
}
```
##### 2）高级特性
- **可中断锁**​：通过 `lockInterruptibly()` 方法响应中断
```java
/**  
 * ReentrantLock * 可中断锁实现  
 */  
public class InterruptibleExample {  
  
    private static final ReentrantLock lock = new ReentrantLock();  
  
    public void doTask() throws InterruptedException {  
        lock.lockInterruptibly(); //可响应中断  
        try {  
            System.out.println("Working...");  
            Thread.sleep(5000); //模拟耗时操作  
        } finally {  
            lock.unlock();  
        }  
    }  
  
    public static void main(String[] args) {  
        InterruptibleExample example = new InterruptibleExample();  
        Thread worker = new Thread(new Runnable() {  
            public void run() {  
                try {  
                    example.doTask();  
                } catch (InterruptedException e) {  
                    System.out.println("线程被中断！");  
                }  
            }  
        });  
        lock.lock();//主线程先占用锁  
        worker.start();  
        worker.interrupt(); //中断worker 线程  
        lock.unlock();  
    }  
  
}
```
- ​**超时锁**​：使用 `tryLock(long timeout, TimeUnit unit)` 避免无限等待
```java
/**  
 * ReentrantLock * 限时锁定  
 */  
public class TimeoutExample {  
    private static ReentrantLock lock = new ReentrantLock();  
  
    public void attemptTask() throws InterruptedException {  
        if (lock.tryLock(1, TimeUnit.SECONDS)){  
            try {  
                System.out.println("锁获取成功");  
            } finally {  
                lock.unlock();  
            }  
        } else {  
            System.out.println("锁获取超时");  
        }  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        TimeoutExample example = new TimeoutExample();  
        example.attemptTask();  
    }  
}
```
- ​**条件变量（Condition）​**​：支持多个等待队列，适用于生产者-消费者模型
```java
/**  
 * ReentrantLock * 条件变量 Condition  
 * 生产者消费者模型  
 */  
public class ConditionExample {  
  
    private final ReentrantLock lock = new ReentrantLock();  
    private final Condition notEmpty = lock.newCondition(); //条件变量1  
    private final Condition notFull = lock.newCondition(); //条件变量2  
    private Queue<Integer> queue = new LinkedList<Integer>();  
    private int capacity = 10;  
  
    public void produce(int item) throws InterruptedException {  
        lock.lock();  
        try {  
            while (queue.size() == capacity) {  
                notFull.await(); //队列满时等待  
            }  
            queue.add(item);  
            notEmpty.signal(); //唤醒消费者  
        } finally {  
            lock.unlock();  
        }  
    }  
  
    public int consume() throws InterruptedException {  
        lock.lock();  
        try {  
            while (queue.isEmpty()) {  
                notEmpty.await(); //队列空时等待  
            }  
            int item = queue.poll();  
            notFull.signal();//唤醒生产者  
            return item;  
        } finally {  
            lock.unlock();  
        }  
    }  
}
```
- 公平锁：实现公平锁，按顺序执行
```java
/**  
 * ReentrantLock * 公平锁  
 */  
public class FairExample {  
  
    ReentrantLock fairLock = new ReentrantLock(true);  
  
    public void fairLock() {  
        fairLock.lock();  
        try {  
            System.out.println("按请求顺序分配锁");  
        } finally {  
            fairLock.unlock();  
        }  
    }  
  
}
```
可重入性：同个线程可多次获取锁
```java
/**  
 * Reentrant * 可重入  
 */  
public class ReentrantExample {  
    private final ReentrantLock lock = new ReentrantLock();  
  
    public void outer(){  
        lock.lock();  
        try {  
            System.out.println("Outer method");  
            inner(); //嵌套调用锁方法  
        } finally {  
            lock.unlock();  
        }  
    }  
  
    public void inner(){  
        lock.lock(); //同一线程可重入  
        try {  
            System.out.println("Inner method");  
        } finally {  
            lock.unlock();  
        }  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        ReentrantExample example = new ReentrantExample();  
        example.outer();  
  
        Thread t1 = new Thread(() -> example.outer()); 
        t1.start();  
        t1.join();   
        System.out.println("可重入锁 ");  
    }  
}
```
#### 四、注意事项
1. ​**必须手动释放锁**​：忘记 `unlock()` 会导致死锁或资源泄漏
2. ​**避免嵌套死锁**​：确保加锁次数与释放次数严格匹配
3. ​**合理选择锁类型**​：
    - 高并发场景优先选择非公平锁（默认）
    - 严格顺序需求（如银行排队）使用公平锁
4. ​**避免忙等（Busy-Waiting）​**​：使用 `tryLock()` 时需设置合理超时时间，否则可能导致 CPU 空转
5. ​**条件变量的正确使用**​：调用 `await()`/`signal()` 前必须持有锁

#### 五、面试核心考点
1. ​`AQS` 原理：`CLH` 队列、state 状态管理、线程唤醒机制
2. ​**公平锁与非公平锁的区别**​：实现差异（队列插队逻辑）、性能对比
3. ​**可重入性实现细节**​：state 计数、`exclusiveOwnerThread` 的作用
4. ​**与 synchronized 的对比**​：
    - ​**功能**​：`ReentrantLock` 支持更多特性（可中断、超时、多条件变量）
    - ​**性能**​：高竞争场景下 `ReentrantLock` 更优
5. ​**Condition 的应用**​：如何实现线程间协作（如生产者-消费者）

#### 六、适用场景
1. ​**需要公平锁**​：如银行排队系统、资源限流
2. ​**响应中断的场景**​：如任务调度系统、数据库连接池
3. ​**多条件变量控制**​：生产者-消费者模型（不同条件队列分离读写线程）
4. ​**手动锁管理**​：需要精确控制锁的获取和释放时机，避免异常导致死锁
5. ​**超时锁需求**​：高并发请求控制，避免线程无限等待
