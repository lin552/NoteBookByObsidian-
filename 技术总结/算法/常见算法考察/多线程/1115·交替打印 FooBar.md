---
创建时间: 2025-06-09 19:00:31
作者: wangxiaoming
tags:
  - 生产者消费者
  - 算法
---
#### 一、题目
给你一个类：

class FooBar {
  public void foo() {
    for (int i = 0; i < n; i++) {
      print("foo");
    }
  }

  public void bar() {
    for (int i = 0; i < n; i++) {
      print("bar");
    }
  }
}

两个不同的线程将会共用一个 `FooBar` 实例：

- 线程 A 将会调用 `foo()` 方法，而
- 线程 B 将会调用 `bar()` 方法

请设计修改程序，以确保 `"foobar"` 被输出 `n` 次。

**示例 1：**

**输入：**n = 1
**输出：**"foobar"
**解释：**这里有两个线程被异步启动。其中一个调用 foo() 方法, 另一个调用 bar() 方法，"foobar" 将被输出一次。

**示例 2：**

**输入：**n = 2
**输出：**"foobarfoobar"
**解释：**"foobar" 将被输出两次。

**提示：**

- `1 <= n <= 1000`

#### 二、解题思路
1.使用阻塞队列
2.`CyclicBarrier`
3.`volatile + CPU` 让出资源
4.`ReentrantLock + Condition`
5.`Synchronized + 标志位`

#### 三、代码
```java
// 1.阻塞队列
class FooBar {
    private int n;
    private BlockingQueue<Integer> bar = new LinkedBlockingQueue<>(1);
    private BlockingQueue<Integer> foo = new LinkedBlockingQueue<>(1);

    public FooBar(int n) {
        this.n = n;
    }

    public void foo(Runnable printFoo) throws InterruptedException {

        for (int i = 0; i < n; i++) {
            foo.put(1);
            printFoo.run();
            bar.put(1);
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            bar.take();
            printBar.run();
            foo.take();
        }
    }
}

//2.CyclicBarrier
class FooBar {
    private int n;
    public FooBar(int n) {
        this.n = n;
    }
    
    CyclicBarrier cb = new CyclicBarrier(2);
    volatile boolean fin = true;
    
    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            while(!fin);
            printFoo.run();
            fin = false;
            try {
                cb.await();
            } catch (BrokenBarrierException e){}
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            try {
                cb.await();
            } catch (BrokenBarrierException e){}
            printBar.run();
            fin = true;
        }
    }
}

//3.volatile可见标记 + 让出CPU 
class FooBar {

    private int n;
    public FooBar(int n) {
        this.n = n;
    }
    
    volatile int flag;

    public void foo(Runnable printFoo) throws InterruptedException {
        int i = 0;
        while(i < n) {
           if(flag == 0){
              printFoo.run();
              flag = 1;
              i++;
           } else {
              Thread.yield();
           }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        int i = 0;
        while(i < n){
           if(flag == 1){
              printBar.run();
              flag = 0;
              i++;
           }else{
              Thread.yield();
           }
        }
    }
}

//4.ReentrantLock + Condition
class FooBar { 
    private int n;
    public FooBar(int n) {
        this.n = n;
    }

    Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private int flag;

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            lock.lock();
            try {
                while(flag == 1){
                    condition.await();
                }
                flag = 1;
                printFoo.run();
                condition.signal();
            } finally {
                lock.unlock();
            }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            lock.lock();
            try {
                while(flag == 0){
                condition.await();
                }
                flag = 0;
                printBar.run();
                condition.signal();
            } finally {
                lock.unlock();
            }
        }
    }
}

//5.Synchronized + 标志位
class FooBar {
    private int n;
    private final Object lock = new Object(); //锁标志
    private int flag;

    public FooBar(int n) {
        this.n = n;
    }

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            synchronized(lock){
                while(flag == 1){
                    lock.wait();
                }
                printFoo.run();
                flag = 1;
                lock.notifyAll();
            }
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            synchronized(lock){
                while(flag == 0){
                    lock.wait();
                }
                printBar.run();
                flag = 0;
                lock.notifyAll();
            }
        }
    }
}

//6.信号量 Semaphore
class FooBar {

    private int n;
    private final Semaphore semaphore1;
    private final Semaphore semaphore2;
    
    public FooBar(int n) {
        this.n = n;
        this.semaphore1 = new Semaphore(1);
        this.semaphore2 = new Semaphore(0);
    }

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            semaphore1.acquire();
            printFoo.run();
            semaphore2.release();
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            semaphore2.acquire();
            printBar.run();
            semaphore1.release();
        }
    }
}
```