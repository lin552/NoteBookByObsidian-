---
创建时间: 2025-07-01 23:16:14
作者: wangxiaoming
tags:
  - 线程
---
在Java多线程编程中，`Thread.sleep()`和`Object.wait()`都是用于控制线程执行流程的方法，但它们的设计目的、调用方式和行为有本质区别。以下从**核心区别**、**底层机制**和**使用场景**三个维度详细对比，并结合代码示例说明：

#### 一、核心区别总结

|​**维度**​|​**Thread.sleep(long millis)​**​|​**Object.wait(long timeout)​**​|
|---|---|---|
|​**方法归属**​|`Thread`类的静态方法（`Thread.sleep()`）。|`Object`类的实例方法（`object.wait()`）。|
|​**调用条件**​|无锁要求：可在任意线程中调用（无需持有锁）。|必须持有对象的**监视器锁**​（`synchronized`同步块/方法中调用）。|
|​**锁的释放**​|不释放锁：调用`sleep()`时，若当前线程持有锁（如在`synchronized`块中），锁不会被释放。|释放锁：调用`wait()`时，会主动释放当前持有的对象的监视器锁，允许其他线程获取锁。|
|​**唤醒条件**​|时间到期自动唤醒；或被其他线程调用`interrupt()`中断（抛出`InterruptedException`）。|其他线程调用`notify()`/`notifyAll()`唤醒；或时间到期自动唤醒；或被中断（抛出`InterruptedException`）。|
|​**作用对象**​|作用于当前线程（暂停当前线程的执行）。|作用于调用该方法的线程（使当前线程进入等待状态，直到被唤醒）。|
|​**使用场景**​|线程休眠（如模拟耗时操作、控制执行节奏）。|线程间通信（如生产者-消费者模型中，消费者等待生产者通知）。|
#### 二、底层机制与行为差异
##### 1. ​**锁的持有与释放**​
- ​**`Thread.sleep()`**​：  
    调用`sleep()`时，线程会进入`TIMED_WAITING`状态，但**不会释放当前持有的任何锁**。即使线程在`synchronized`同步块中调用`sleep()`，其他线程仍无法获取该锁，直到`sleep()`结束。
```java
public class SleepDemo {
    private static final Object lock = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("线程A持有锁，开始sleep...");
                    Thread.sleep(3000); // 不释放锁
                    System.out.println("线程A sleep结束，释放锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(() -> {
            synchronized (lock) { // 等待线程A释放锁（3秒后才能进入）
                System.out.println("线程B获取到锁");
            }
        }).start();
    }
}
```
输出：
```
线程A持有锁，开始sleep...
（3秒后）
线程A sleep结束，释放锁
线程B获取到锁
```
​
**`Object.wait()`**​：  
调用`wait()`时，线程会释放当前持有的对象的监视器锁，并进入`WAITING`或`TIMED_WAITING`状态。其他线程可以获取该锁并执行同步块中的代码。
```java
public class WaitDemo {
    private static final Object lock = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("线程A持有锁，开始wait...");
                    lock.wait(3000); // 释放锁，进入等待
                    System.out.println("线程A被唤醒，重新获取锁");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(() -> {
            synchronized (lock) { // 立即获取锁（线程A已释放）
                System.out.println("线程B获取到锁，执行操作...");
                lock.notify(); // 唤醒线程A
            }
        }).start();
    }
}
```
输出：
```
线程A持有锁，开始wait...
线程B获取到锁，执行操作...
线程A被唤醒，重新获取锁
```
##### 2. ​**唤醒条件**​
- ​**`Thread.sleep()`**​：  
    只能通过两种方式唤醒：
    - 时间到期（自动唤醒）；
    - 其他线程调用当前线程的`interrupt()`方法（抛出`InterruptedException`）。
    
- ​**`Object.wait()`**​：  
    可以通过三种方式唤醒：
    - 其他线程调用`notify()`（随机唤醒一个等待该锁的线程）；
    - 其他线程调用`notifyAll()`（唤醒所有等待该锁的线程）；
    - 时间到期（自动唤醒）；
    - 被中断（抛出`InterruptedException`）。

##### 3. ​**异常处理**​
两者均声明了`InterruptedException`，但触发场景不同：
- `sleep()`的中断：当线程在`sleep()`时被调用`interrupt()`，会立即抛出`InterruptedException`，并清除中断状态；
- `wait()`的中断：当线程在`wait()`时被调用`interrupt()`，会抛出`InterruptedException`，并重新获取锁（若中断发生在`wait()`返回前）。

#### 三、使用场景对比
|​**方法**​|​**典型场景**​|
|---|---|
|`Thread.sleep()`|- 模拟耗时操作（如网络请求、文件IO）；  <br>- 控制线程执行节奏（如定时任务）；  <br>- 测试多线程竞争（如强制线程切换）。|
|`Object.wait()`|- 线程间通信（如生产者-消费者模型中，消费者等待队列非空）；  <br>- 协调多线程执行顺序（如线程A等待线程B完成初始化）。|
#### **总结**​
- ​**核心区别**​：`sleep()`是线程的“主动休眠”（不释放锁），`wait()`是线程的“协作等待”（释放锁）；
- ​**锁行为**​：`sleep()`不释放锁，`wait()`释放锁；
- ​**唤醒方式**​：`sleep()`依赖时间或中断，`wait()`依赖`notify()`/`notifyAll()`或中断；
- ​**设计目的**​：`sleep()`控制线程执行节奏，`wait()`实现线程间同步。