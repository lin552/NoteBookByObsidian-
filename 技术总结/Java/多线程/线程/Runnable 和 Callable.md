---
创建时间: 2025-04-12 17:34:32
作者: wangxiaoming
tags:
  - 多线程
  - Thread
  - Java
---

#### `Runnable` 接口

##### 特性
- ​**无返回值**：`run()` 方法返回 `void`，适用于不需要返回结果的任务
- ​**异常处理**：无法直接抛出受检异常（checked exceptions），需在方法内部处理
- ​**直接支持**：可通过 `Thread` 类直接启动，或提交给线程池执行

```java
// 实现 Runnable 接口
class PrintTask implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 执行任务");
    }
}

public class Main {
    public static void main(String[] args) {
        // 方式1：通过 Thread 启动
        Thread thread = new Thread(new PrintTask());
        thread.start();

        // 方式2：提交到线程池（如 ExecutorService）
        ExecutorService executor = Executors.newSingleThreadExecutor();
        executor.execute(new PrintTask());
        executor.shutdown();
    }
}
```


#### `Callable` 接口
##### 特性
- **有返回值**：`call()` 方法返回泛型结果（如 `Integer`、`String`）
- ​**异常处理**：可声明抛出受检异常（如 `IOException`）
- ​**间接支持**：需通过 `FutureTask` 包装后提交给 `Thread` 或线程池

```java
import java.util.concurrent.*;

class ComputeTask implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 0; i < 100; i++) {
            sum += i;
        }
        return sum;
    }
}

public class Main {
    public static void main(String[] args) throws Exception {
        // 包装为 FutureTask
        FutureTask<Integer> futureTask = new FutureTask<>(new ComputeTask());
        
        // 方式1：通过 Thread 启动
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println("计算结果：" + futureTask.get()); // 阻塞获取结果

        // 方式2：提交到线程池（推荐）
        ExecutorService executor = Executors.newCachedThreadPool();
        Future<Integer> future = executor.submit(new ComputeTask());
        System.out.println("计算结果：" + future.get());
        executor.shutdown();
    }
}
```