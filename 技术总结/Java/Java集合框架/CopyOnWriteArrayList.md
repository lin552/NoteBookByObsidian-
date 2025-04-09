---
创建时间: 2025-03-14T18:39:00
作者: wangxiaoming
tags:
  - 集合
  - ArryList
---
CopyOnWriteArrayList 是 Java 并发包（`java.util.concurrent`）中实现的高效线程安全集合类，专为 ​**读多写少** 的并发场景设计。

#### 一、核心原理与设计思想
##### 1）写时复制（Copy-On-Write）机制
- **操作流程**：
    - ​**写操作**​（如 `add`、`remove`）：通过加锁（`ReentrantLock`）复制当前数组，在新数组上完成修改后，将原数组引用替换为新数组。
    - ​**读操作**：直接访问原数组，无需加锁。
- ​**目的**：保证 ​**读操作完全无锁**，实现读写分离，最大化读性能。

##### 2）线程安全实现
- ​**写操作锁**：通过 `ReentrantLock` 保证同一时刻只有一个线程执行写操作，避免多线程并发写导致数据混乱。
- ​**可见性**：使用 `volatile` 修饰底层数组引用，确保数组替换后其他线程能立即感知到新数据。

##### 3）弱一致性迭代器
- ​**快照机制**：迭代器基于创建时的数组快照遍历数据，不反映后续修改。
- ​**无 `ConcurrentModificationException`**：由于迭代器与写操作操作不同数组，避免了快速失败（fail-fast）机制的问题。

#### 二、数据结构与源码实现
##### 1)底层结构
- ​**动态数组**：数据存储在 `volatile Object[] array` 中，初始为空数组。
- ​**扩容逻辑**：每次写操作都会创建新数组（长度+1），旧数组被 GC 回收。
##### 2)关键方法实现
###### add方法
```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        Object[] newElements = Arrays.copyOf(elements, elements.length + 1);
        newElements[elements.length] = e;
        setArray(newElements); // volatile 写保证可见性
        return true;
    } finally {
        lock.unlock();
    }
}
```
**`get()` 方法**：直接访问 `array` 数组，无锁、无同步。

#### 三、适用场景与性能分析
1. ​**最佳场景**
    - ​**高频读、低频写**：如缓存系统、配置管理、事件监听器列表。
    - ​**弱一致性需求**：允许读操作获取旧数据，但保证最终一致性。
2. ​**性能特点**
    - ​**读性能**：接近 `ArrayList`（O(1) 时间复杂度），无锁且无阻塞。
    - ​**写性能**：较差（需复制数组），适合数据量小（千级以内）的场景。
    - ​**内存开销**：频繁写操作会导致内存占用高，可能触发多次 GC。
3. ​**与ArrayList对比**

| ​**维度**    | `CopyOnWriteArrayList` | `ArrayList`  |
| ---------- | ---------------------- | ------------ |
| ​**线程安全**  | 是（写操作加锁）               | 否            |
| ​**读性能**   | 高（无锁）                  | 高（无锁）        |
| ​**写性能**   | 低（需复制数组）               | 高（直接修改数组）    |
| ​**内存效率**  | 低（频繁写时内存占用高）           | 高            |
| ​**迭代一致性** | 弱一致性（快照）               | 强一致性（可能抛出异常） |
#### 四、注意事项与替代方案
1. ​**使用建议**
    - ​**避免大数据量频繁写**：数组复制成本高，可能导致性能瓶颈。
    - ​**内存监控**：大量写操作需关注 GC 压力，避免内存泄漏。
2. ​**替代方案**
    - ​**`ConcurrentLinkedQueue`**：适用于高并发队列场景（无锁 CAS 实现）。
    - ​**`Collections.synchronizedList`**：简单同步包装类，适合低并发场景。
    - ​**读写锁（[[ReentrantReadWriteLock]]）​**：手动控制读写分离，灵活性更高。