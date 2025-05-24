---
创建时间: 2025-04-20 11:23:16
作者: wangxiaoming
tags:
  - Semaphore
---
#### 一、什么是`Semaphore`?
Semaphore（信号量）是 Java 并发包（`java.util.concurrent`）中的一种同步工具，用于控制对共享资源的并发访问。它通过“许可证”机制，限制同时访问资源的线程数量，常用于流量控制、资源池管理等场景
#### 二、核心原理
- ​**基于 `AQS` 实现**​：Semaphore 内部通过 `AbstractQueuedSynchronizer`（`AQS`）维护许可证计数器（`state`变量），实现线程阻塞与唤醒 [[AQS (AbstractQueuedSynchronizer)]]
- ​**许可证管理**​：
    - ​**获取（acquire）​**​：线程尝试减少计数器值，若计数器≥0则成功；若<0则线程进入阻塞队列等待
    - ​**释放（release）​**​：线程增加计数器值，并唤醒等待队列中的线程
- ​**公平性**​：
    - ​**非公平模式（默认）​**​：允许新请求插队，可能导致饥饿但吞吐量高
    - ​**公平模式**​：按请求顺序分配许可证，避免饥饿但性能较低
#### 三、用法
##### 1）基础用法
```java
Semaphore semaphore = new Semaphore(10); //初始化10个许可证
//基础获取与释放
semaphore.acquire();//阻塞获取1个许可证
try {
   // 访问共享资源
} finally {
   semaphore.release(); //释放许可证
}
```
##### 2）高级用法：
- ​**批量获取/释放**​：`acquire(int permits)` 和 `release(int permits)`
- ​**非阻塞尝试**​：`tryAcquire()` 或带超时的 `tryAcquire(long timeout, TimeUnit unit)`
- ​**动态调整许可证**​：通过 `reducePermits()` 或 `drainPermits()` 调整资源容量
##### 3) 用法示例
```java
//停车场车位控制（10个车位）
Semaphore parkingLot = new Semaphore(10);
//车辆进入
parkingLot.acquire();
try {
   System.out.println("车辆进入，剩余车位："+parkingLot.availablePermits())
} finally {
  parkingLot.release
}
```
#### 四、使用注意事项
1. ​**许可证泄漏**​：确保 `acquire()` 后必有 `release()`，建议使用 `try-finally` 块
2. ​**死锁风险**​：避免多个信号量相互等待，例如线程A持有`S1`等待`S2`，线程B持有`S2`等待`S1`
3. ​**超时设置**​：优先使用带超时的 `tryAcquire()`，防止线程无限阻塞
4. ​**公平性选择**​：根据场景权衡吞吐量与公平性，非公平模式性能更优
#### 五、性能优化
- ​**合理初始值**​：根据系统负载设置许可证数量，避免过度竞争或资源浪费
- ​**监控与调参**​：观察吞吐量、响应时间等指标，动态调整许可证数
- ​**替代方案**​：单进程场景优先使用轻量级 `SemaphoreSlim`（非Java原生），或结合线程池管理并发
#### 六、面试考点
1. ​**原理实现**​：基于AQS的计数器、公平/非公平模式区别
2. ​**死锁与泄漏**​：如何避免许可证未释放或循环等待
3. ​**行为分析**​：
    - 初始化10许可证，11线程同时 `acquire()`：1线程阻塞
    - 线程多次 `acquire()`：可能导致计数器为负，线程阻塞
4. ​**特殊场景**​：`release()` 后计数器超过初始值是否允许？允许，但需注意溢出风险

#### 七、适用场景
1. ​**资源池管理**​：如数据库连接池，限制并发连接数
2. ​**接口限流**​：控制单位时间内API调用量，防止服务过载
3. ​**生产者-消费者模型**​：协调生产与消费速率，例如有界队列
4. ​**互斥锁替代**​：设置许可证为1时，可替代 `ReentrantLock`（但不可重入）
