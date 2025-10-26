---
创建时间: 2025-06-05 23:07:52
作者: wangxiaoming
tags:
  - Exchanger
  - CAS
---
Java 中的 ​**`Exchanger`**​ 是 `java.util.concurrent` 包下的同步工具类，用于**两个线程之间的直接数据交换**。它通过一个同步点（Exchange Point）实现两个线程的协作，允许线程在到达该点时交换各自持有的数据。其核心价值在于**简化双线程协作的逻辑**，适用于需要双线程数据交换的场景（如生产者-消费者、任务协作等）。以下是其核心考点的详细梳理：

#### 一、核心功能与设计思想
`Exchanger` 的核心是一个**同步交换点**，两个线程通过调用 `exchange()` 方法在该点相遇并交换数据。其核心特性包括：
- ​**双线程协作**​：仅支持两个线程之间的交换（多线程需配合其他机制）。
- ​**数据交换**​：线程 A 持有数据 `A`，线程 B 持有数据 `B`，交换后线程 A 得到 `B`，线程 B 得到 `A`。
- ​**阻塞等待**​：若线程到达交换点时对方未到达，当前线程会阻塞，直到对方到达并完成交换。

#### 二、核心方法解析
`Exchanger` 提供了两个核心方法：
##### 1. `exchange(V x)`：无超时交换​
- ​**作用**​：当前线程携带数据 `x` 到达交换点，阻塞等待另一个线程到达并交换数据。
- ​**返回值**​：另一个线程携带的数据（交换成功后返回）。
- ​**示例**​：
    ```java
    Exchanger<String> exchanger = new Exchanger<>();
    
    // 线程 A
    new Thread(() -> {
        try {
            String dataA = "Hello";
            String dataFromB = exchanger.exchange(dataA); // 阻塞等待线程 B
            System.out.println("线程 A 收到：" + dataFromB);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
    
    // 线程 B
    new Thread(() -> {
        try {
            String dataB = "World";
            String dataFromA = exchanger.exchange(dataB); // 阻塞等待线程 A
            System.out.println("线程 B 收到：" + dataFromA);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
    ```

    ```markdown
    线程 A 收到：World
    线程 B 收到：Hello
    ```
##### 2. `exchange(V x, long timeout, TimeUnit unit)`：带超时的交换​
- ​**作用**​：当前线程携带数据 `x` 到达交换点，最多阻塞 `timeout` 时间。若超时仍未等到对方线程，交换失败并抛出 `TimeoutException`。
- ​**返回值**​：交换成功时返回对方数据；超时返回 `null`（或已持有的数据，取决于实现）。

#### 三、底层实现原理
`Exchanger` 底层通过 ​**CAS（Compare-And-Swap）​**​ 和 ​**等待队列**​ 实现线程的阻塞与唤醒，核心逻辑如下：
##### ​1. 状态管理​
- ​**`slot` 变量**​：用于存储当前等待线程的数据（初始为 `null`）。
- ​**`waiting` 标志**​：标记是否有线程正在等待交换（避免重复唤醒）。
##### ​**2. 交换流程**​
1. ​**线程 A 到达交换点**​：
    - 检查 `slot` 是否为 `null`（无等待线程）。
    - 若为 `null`，将自身数据存入 `slot`，并阻塞等待（通过 `LockSupport.park()`）。
    
2. ​**线程 B 到达交换点**​：
    - 检查 `slot` 是否非 `null`（有线程 A 等待）。
    - 若非 `null`，获取线程 A 的数据（`slot`），将自身数据存入 `slot`，并唤醒线程 A。
    
3. ​**线程 A 被唤醒**​：
    - 从 `slot` 中获取线程 B 的数据，交换完成。

#### 四、典型使用场景
##### ​1. 生产者-消费者模型​
两个线程分别作为生产者和消费者，交换生产的数据和消费的确认。例如：
```java
Exchanger<Product> exchanger = new Exchanger<>();

// 生产者线程
new Thread(() -> {
    while (true) {
        Product product = produce(); // 生产产品
        Product confirmation = exchanger.exchange(product); // 交换为消费确认
        processConfirmation(confirmation); // 处理确认
    }
}).start();

// 消费者线程
new Thread(() -> {
    while (true) {
        Product confirmation = prepareConfirmation(); // 准备确认
        Product product = exchanger.exchange(confirmation); // 交换为产品
        consume(product); // 消费产品
    }
}).start();
```

##### ​2. 双线程任务协作​
两个线程分别完成不同阶段的任务，交换中间结果。例如：
- 线程 A 处理数据的前半部分，线程 B 处理后半部分，交换后合并结果。

##### ​3. 数据校验与处理​
一个线程生成数据，另一个线程校验数据，交换后生成最终结果。

#### 五、关键问题与注意事项
##### 1. 仅支持双线程交换​
`Exchanger` 只能用于两个线程之间的交换，多线程场景需结合其他工具（如 `CyclicBarrier`）。

##### ​2. 阻塞与超时​
- `exchange()` 默认无限阻塞，需注意线程中断（`InterruptedException`）。
- 带超时的 `exchange()` 可避免无限等待（如对方线程崩溃时，当前线程可超时后执行降级逻辑）。

##### ​3. 数据一致性​
交换的数据是线程本地内存中的副本，需确保数据的可见性（Java 内存模型保证 `exchange()` 的原子性，无需额外同步）。

##### ​4. 不可重入性​
同一线程无法多次调用 `exchange()`（否则会阻塞自己）。

#### 六、与`CyclicBarrier`的对比
| ​**特性**​    | ​**Exchanger**​   | ​**CyclicBarrier**​ |
| ----------- | ----------------- | ------------------- |
| ​**协作线程数**​ | 仅 2 个             | 任意多个                |
| ​**交换数据**​  | 支持（双线程数据交换）       | 不支持（仅同步等待）          |
| ​**触发条件**​  | 双线程同时到达交换点        | 所有线程到达屏障点           |
| ​**适用场景**​  | 双线程协作（如数据交换、任务接力） | 多线程同步（如多阶段任务、初始化）   |
#### 七、常见面试题
1. ​`Exchanger` 的核心作用是什么？​**​  
    用于两个线程之间的直接数据交换，通过同步点实现双线程协作。
    
2. ​`exchange()` 方法的两种形式有何区别？​**​  
    无超时版本会无限阻塞直到交换成功；带超时版本在指定时间内未交换成功则抛出 `TimeoutException`。
    
3. ​`Exchanger` 如何保证线程安全？​**​  
    底层通过 `CAS` 操作原子性地更新 `slot` 变量，并结合 `LockSupport.park()`/`unpark()` 实现线程阻塞与唤醒，确保多线程下的可见性和原子性。
    
4. ​`Exchanger` 适合哪些场景？​**​  
    双线程数据交换（如生产者-消费者、任务协作）、需要简化双线程同步逻辑的场景。
    
5. ​`Exchanger` 与 `CyclicBarrier` 的主要区别是什么？​**​  
    `Exchanger` 专注于双线程数据交换，`CyclicBarrier` 专注于多线程同步等待。