---
创建时间: 2025-04-19 21:52:39
作者: wangxiaoming
tags:
  - StampedLock
---
#### 一、什么是`StampedLock`?
`StampedLock` 是 Java 8 引入的高性能锁机制，支持三种访问模式：​**写锁（独占锁）​**、**悲观读锁（共享锁）​**和**乐观读锁（非阻塞读）​**​
。它通过**邮戳（Stamp）​**机制管理锁状态，适用于读多写少的高并发场景，相比 `ReentrantReadWriteLock` 性能更高，但复杂度也更高
#### 二、核心原理
- ​**邮戳（Stamp）机制**​：每次获取锁会返回一个 Stamp，用于解锁或验证数据一致性
- ​**三种模式**​：
    - ​**写锁**​：独占模式，阻塞其他所有锁请求
    - ​**悲观读锁**​：共享模式，允许多个线程同时读，但阻塞写锁请求
    - ​**乐观读锁**​：无需加锁，读取数据后通过 `validate()` 验证是否被修改，失败则升级为悲观读锁
- ​**底层实现**​：基于 `CAS` 操作和自旋优化，内部维护等待队列，优先处理写锁请求以避免饥饿
#### 三、基础与高级用法
##### 1）基础用法
###### 写锁
```java
long stamp = lock.writeLock();
try { /* 写操作 */ }
finally { lock.unlockWrite(stamp); }
```
###### 悲观读锁
```java
long stamp = lock.readLock();
try { /* 读操作 */ }
finally { lock.unlockRead(stamp); }
```
###### 乐观读锁
```java
long stamp = lock.tryOptimisticRead();
//读取数据到局部变量
if(!lock.validate(stamp)){
   stamp = lock.readLock();
   try { /* 重新读取 */ }
   finally{ lock.unlockRead(stamp); }
}
```
##### 2）高级用法
###### 锁转换
支持从读锁升级为写锁（需验证是否成功）
```java
long stamp = lock.readLock();
try {
    long ws = lock.tryConvertToWriteLock(stamp);
    if (ws != 0L) { stamp = ws; /* 执行写操作*/}
} finally { lock.unlock(stamp); }
```
##### 3)综合案例
假设有一个银行账户类 `BankAccount`，需要支持以下操作：
- ​**存款（写锁）​**​：线程安全地修改余额。
- ​**悲观读余额（悲观读锁）​**​：确保读取时无写操作。
- ​**乐观读余额（乐观读锁）​**​：高效读取余额，允许写操作。
- ​**余额为0时提现（锁转换）​**​：从读锁升级为写锁。

```java
public class BankAccount {  
    private double balance;  
    private final StampedLock lock  = new StampedLock();  
  
    //存款（写锁）  
    public void deposit(double amount) {  
        long stamp = lock.writeLock(); //获取写锁  
        try {  
            balance += amount;  
            System.out.println("存款成功，余额："+balance);  
        } finally {  
            lock.unlockWrite(stamp); //释放写锁  
        }  
    }  
  
    //悲观读余额（悲观读锁）  
    public double getBalancePessimistic() {  
        long stamp = lock.readLock();//获取悲观读锁  
        try {  
            System.out.println("悲观读余额："+balance);  
            return balance;  
        } finally {  
            lock.unlockRead(stamp); //释放读锁  
        }  
    }  
  
    //乐观读余额（乐观读锁）  
    public double getBalanceOptimistic() {  
        long stamp = lock.tryOptimisticRead(); //尝试乐观读  
        double currentBalance = balance; //读取到本地变量  
        //检查期间是否有写操作(例如：存款)  
        if (!lock.validate(stamp)) { //验证失败  
            System.out.println("乐观读失败，升级为悲观读");  
            stamp = lock.readLock(); //升级为悲观读锁  
            try {  
                currentBalance = balance;  
            } finally {  
                lock.unlockRead(stamp);  
            }  
        }  
        System.out.println("乐观读余额："+currentBalance);  
        return currentBalance;  
    }  
  
    //余额为0时提现（锁转换）  
    public void withdrawIfZero(double amount) {  
        long stamp = lock.readLock(); //先获取读锁  
        try {  
            while(balance == 0){  
                long ws = lock.tryConvertToWriteLock(stamp);  
                if(ws != 0L){ //转换成功  
                    stamp = ws; //更新stamp为写锁  
                    balance -= amount;  
                    System.out.println("提现成功，余额"+balance);  
                    break;  
                } else { //转换成功  
                    lock.unlockRead(stamp); //显示释放读锁  
                    stamp = lock.writeLock(); //直接获取写锁  
                }  
            }  
        } finally {  
            lock.unlock(stamp);  
        }  
    }  
  
    public static void main(String[] args) {  
        BankAccount account = new BankAccount();  
        account.deposit(0);  
        account.withdrawIfZero(10);  
  
    }  
}
```

#### 四、注意事项
- ​**不可重入**​：同一线程重复获取锁会导致死锁（需避免嵌套锁操作）
- ​**正确释放锁**​：必须使用对应 Stamp 解锁，否则抛出异常
- ​**中断处理**​：阻塞时若线程被中断，可能导致 CPU 飙升（需显式处理中断）
- ​**不支持条件变量**​：无法使用 `Condition` 类
- ​**数据一致性**​：乐观读需在验证失败后重新读取

#### 五、优化建议
- ​**优先使用乐观读**​：减少锁竞争，适合短时读操作
- ​**避免长时间持有锁**​：防止写锁饥饿，提升吞吐量
- ​**合理降级**​：验证失败时及时降级为悲观读锁，确保数据正确
- ​**避免锁转换死锁**​：转换失败需显式释放锁再重试

#### 六、面试常见问题
1. ​**与 `ReentrantReadWriteLock` 的区别？​**​
    - `StampedLock` 支持乐观读、不可重入、性能更高；`ReentrantReadWriteLock` 支持重入和条件变量
2. ​**乐观读的实现原理？​**​
    - 通过 Stamp 版本号验证数据是否被修改
3. ​**如何解决写线程饥饿问题？​**​
    - 优先处理写锁请求，非公平模式下允许写锁插队
4. ​**死锁场景举例？​**​
    - 线程重复获取写锁（不可重入）或未正确释放锁

#### 七、适用场景
- ​**读多写少**​：如缓存系统、实时数据统计（高频读，低频写）
- ​**高性能需求**​：需要减少锁竞争的场景（如金融高频交易）
- ​**复杂锁转换**​：需要动态调整锁模式（如读后写）的业务逻辑
