---
创建时间: "2025-08-25 18:40:21"
作者: wangxiaoming
tags:
---
#### 一、锁机制：互斥锁与读写锁结合
##### 1.互斥锁（`Mutex`）
- **作用**​：保护关键资源（如内存映射文件、哈希表）的独占访问，确保同一时间仅一个线程执行写操作。
- **实现**​：
    - 使用 `pthread_mutex_t`实现递归锁（`PTHREAD_MUTEX_RECURSIVE`），允许同一线程多次加锁（避免死锁）。
    - ​**示例**​：在 `setString`和 `getString`方法中，通过 `SCOPED_LOCK(m_lock)`加锁，确保读写操作的原子性
```cpp
bool MMKV::setString(const std::string &key, const std::string &value) {
    SCOPED_LOCK(m_lock);  // 加锁
    // 写入数据
}
```

##### 2.读写锁
- **作用**​：允许多个线程并发读取，但写操作互斥，提升读多写少场景的性能。
- ​**实现**​：
    - 使用 `pthread_rwlock_t`实现读写锁，读操作加读锁（`lockRead`），写操作加写锁（`lockWrite`）。
    - ​**示例**​：在 `getString`中使用读锁，允许多线程同时读取；`setString`使用写锁，确保写操作独占
```cpp
bool MMKV::getString(const std::string &key, std::string &value) {
    SCOPED_READ_LOCK(m_rwLock);  // 加读锁
    // 读取数据
}
```

#### 二、原子操作与内存屏障
##### 1.原子操作
- ​**作用**​：通过轻量级同步保证关键变量（如引用计数、状态标志）的线程安全。
- **实现**​：
    - 使用 C++11 的 `std::atomic`实现原子变量（如 `Atomic<int> m_refCount`）。
    - **示例**​：引用计数通过 `increment()`和 `decrement()`实现无锁增减
```cpp
Atomic<int> m_refCount;  // 引用计数
m_refCount.increment();  // 原子递增
```
##### 2.内存屏障（Memory Barrier）
- ​**作用**​：确保内存操作的顺序性和可见性，防止指令重排序。
- ​**实现**​：
    - 使用 `std::atomic_thread_fence`实现全内存屏障（`memory_order_seq_cst`）。
    - ​**示例**​：在 `flush()`方法中，通过屏障确保内存写入完成后再同步到磁盘
```cpp
MemoryBarrier::release();  // 释放屏障，确保写入完成
fsync(m_fd);               // 同步到磁盘
```

#### 三、多进程同步机制
##### 1.文件锁（File Lock）
- **作用**​：在多进程环境下，通过文件锁保证数据一致性。
- ​**实现**​：
    - 使用 `fcntl`实现文件的排他锁（`F_WRLCK`）和共享锁（`F_RDLCK`）。
    - ​**示例**​：写入数据前获取排他锁，读取时获取共享锁
```cpp
FileLock fileLock(m_fd);
if (fileLock.lockExclusive()) {  // 排他锁
    // 写入数据
}
```

##### 2.内存映射同步
- ​**作用**​：通过 `mmap`将文件映射到内存，结合 `msync`确保内存与磁盘数据一致。
- ​**实现**​：
    - 写入时通过 `msync(m_ptr, m_actualSize, MS_SYNC)`强制同步内存到磁盘

#### 四、动态扩容与锁颗粒度控制
##### 1.动态扩容
- ​**策略**​：采用指数增长策略（如每次扩容为原大小的 2 倍），减少扩容频率。
- ​**线程安全**​：扩容时加锁（`SCOPED_LOCK(m_lock)`），确保扩容期间无其他线程干扰
##### 2.锁颗粒优化
- ​**细粒度锁**​：仅在必要时加锁（如查找键值时），减少锁持有时间。
- ​**示例**​：`containsKey`方法仅在查找时加锁，后续操作在锁外执行
```cpp
bool MMKV::containsKey(const std::string &key) {
    {  // 限定锁作用域
        SCOPED_LOCK(m_lock);
        return findKey(key) != INVALID_OFFSET;
    }
    // 锁在此处释放
}
```

#### 五、无锁数据结构与批量操作
##### 1.无锁哈希表
- **实现**​：通过原子操作实现无锁插入和删除，减少锁竞争。
- ​**示例**​：`MMKVHashMap`使用 `compare_exchange_weak`实现无锁链表节点插入
##### 2.批量操作优化
- ​**批量写入**​：通过一次加锁完成多个键值对的写入，减少锁获取次数。
- ​**示例**​：`batchSet`方法批量写入数据，降低锁开销

#### 六、初始化与销毁的线程安全
##### 1.单例模式
•**实现**​：通过 `std::call_once`和互斥锁确保全局实例的唯一性。
```cpp
static MMKV *defaultMMKV(...) {
    static std::once_flag s_onceFlag;
    std::call_once(s_onceFlag, []{ /* 初始化实例 */ });
}
```

##### 2.资源释放
•**安全销毁**​：在析构函数中加锁，确保关闭文件描述符和内存映射时无其他线程访问

#### 七、性能与安全平衡
- **递归锁优化**​：允许同一线程多次加锁，避免嵌套调用导致的死锁。
- ​**非公平锁策略**​：默认使用非公平锁减少线程切换开销，提升吞吐量