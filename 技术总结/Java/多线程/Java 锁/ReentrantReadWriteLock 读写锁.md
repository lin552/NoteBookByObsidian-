---
创建时间: 2025-04-20 10:35:55
作者: wangxiaoming
tags:
  - ReentrantReadWriteLock
---
#### 一、基本概念与特性
​`ReentrantReadWriteLock`​ 是 Java 并发库中基于 `ReadWriteLock` 接口实现的读写分离锁，​**核心目标是在读多写少的场景下提升并发性能**。其特性包括：
1. ​**读写分离**​：读锁（共享锁）允许多线程并发读取，写锁（独占锁）保证单线程独占写入
2. ​**可重入性**​：同一线程可重复获取读锁或写锁，避免死锁
3. ​**公平性选择**​：支持公平与非公平模式（默认非公平，吞吐量更高）
4. ​**锁降级**​：允许线程先获取写锁，再获取读锁后释放写锁，确保数据一致性
5. ​**互斥规则**​：读读不互斥、读写互斥、写写互斥

#### 二、实现原理
​**核心机制基于 `AQS（AbstractQueuedSynchronizer）`​**​：[[AQS (AbstractQueuedSynchronizer)]]
1. ​**状态拆分**​：
    - ​**state 变量**​（32 位）高 16 位记录读锁持有次数，低 16 位记录写锁重入次数
    - 读锁通过 `sharedCount()` 获取，写锁通过 `exclusiveCount()` 获取。
2. ​**锁获取逻辑**​：
    - ​**读锁**​：无写锁且线程未被阻塞时，通过 `CAS` 增加读计数；支持线程本地缓存（`HoldCounter`）优化性能
    - ​**写锁**​：仅当无读锁且无其他写锁时，`CAS` 更新写计数；支持重入
3. ​**锁降级流程**​：  
    线程先获取写锁 → 修改数据 → 获取读锁 → 释放写锁 → 后续操作通过读锁保证数据可见性

#### 三、基础与高级使用
##### 1）基础用法
```java
ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock,readLock(); //读锁
Lock writeLock = rwLock.writeLock(); //写锁

//读操作
readLock.lock();
try {/* 读取共享数据 */}
finally { readLock.unlock(); }

//写操作
writeLock.lock();
try { /* 修改共享数据 */ }
finally { writeLock.unlock(); }
```
##### 2）高级用法
###### 锁降级示例（缓存更新场景）
```java
writeLock.lock();
try {
   //修改数据
   readLock.lock(); //锁降级关键步骤
} finally {
   writeLock.unlock();
}
try { /* 读取更新后的数据 */ }
finally { readLock.unlock(); }
```
公平性控制：构造函数传入 `true` 启用公平锁，按请求顺序分配资源
##### 3）使用案例
```java
/**  
 * 场景：微服务频繁读取配置（如数据库连接参数），但配置更新较少。原代码使用 synchronized 导致读取操作串行化，高峰期配置中心成为性能瓶颈  
 *  
 * 读写锁分离读/写操作，读操作并发执行，写操作保持原子性  
 *  
 * 效果：  
 * 微服务启动速度提升 40%，配置读取延迟降低 75%；  
 * 更新配置时不影响正常读取，服务稳定性增强  
 */  
public class ConfigurationManager {  
    private Map<String,String> configs = new HashMap<String,String>();  
    private ReadWriteLock rwLock = new ReentrantReadWriteLock();  
  
    public String getConfig(String key) {  
        rwLock.readLock().lock(); //允许并发读取  
        try {  
            System.out.println("Reading config: " + key);  
            return configs.get(key);  
        } finally {  
            rwLock.readLock().unlock();  
        }  
    }  
  
    public  void updateConfig(String key, String value) {  
        rwLock.writeLock().lock(); //独占写入  
        try {  
            System.out.println("Updating config key: " + key + " value: " + value);  
            configs.put(key,value);  
        } finally {  
            rwLock.writeLock().unlock();  
        }  
    }  
  
    public static void main(String[] args) {  
        ConfigurationManager cm = new ConfigurationManager();  
        new Thread(new Runnable() {  
            public void run() {  
                cm.updateConfig("config1","aaaaa");  
            }  
        }).start();  
        new Thread(new Runnable() {  
            public void run() {  
                cm.getConfig("config1");  
            }  
        }).start();  
        new Thread(new Runnable() {  
            public void run() {  
                cm.updateConfig("config2","bbbbb");  
            }  
        }).start();  
        new Thread(new Runnable() {  
            public void run() {  
                cm.getConfig("config2");  
            }  
        }).start();  
        new Thread(new Runnable() {  
            public void run() {  
                cm.updateConfig("config3","ccccc");  
            }  
        }).start();  
    }  
}

/**  
 * 场景：商品信息查询频率远高于更新频率（读写比 100:1），原代码使用 ReentrantLock 导致高并发下读操作互相阻塞，系统响应延迟严重  
 *  
 * 改用读写锁，读操作并行化，写操作保持独占  
 *  
 * 效果：  
 * 吞吐量提升 10 倍，用户查询响应时间降低 60%  
 * 促销高峰期系统稳定性显著增强。  
 */  
public class ProductCache {  
  
    private Map<String,Product> cache = new HashMap<>();  
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();  
    private final Lock readLock = rwLock.readLock();  
    private final Lock writeLock = rwLock.writeLock();  
  
    public Product getProduct(String productId) {  
        readLock.lock(); //多个线程可同时读取  
        try {  
            System.out.println("Reading product " + productId);  
           return cache.get(productId);  
        } finally {  
            readLock.unlock();  
        }  
    }  
  
    public void updataProduct(String productId, Product product) {  
        writeLock.lock(); //写操作独占  
        try {  
            cache.put(productId,product);  
            System.out.println("Product updated to " + productId);  
        } finally {  
            writeLock.unlock();  
        }  
    }  
  
  
    class Product {  
  
    }  
  
    public static void main(String[] args) {  
        ProductCache productCache = new ProductCache();  
  
        new Thread(()->{  productCache.updataProduct("product2", productCache.new Product()); }).start();  
        new Thread(()->{  productCache.updataProduct("product1", productCache.new Product());}).start();  
        new Thread(()->{  productCache.updataProduct("product3", productCache.new Product()); }).start();  
    }  
  
}
```
#### 四、注意事项与优化
**注意事项**​：
1. ​**锁升级限制**​：持有读锁时无法直接获取写锁（需先释放读锁）
2. ​**写锁饥饿**​：非公平模式下，大量读线程可能阻塞写线程，可通过公平锁或限时策略缓解
3. ​**性能权衡**​：读锁不支持 Condition，仅写锁支持
​**优化建议**​：
4. ​**减少锁粒度**​：对数据分段加锁（如 `ConcurrentHashMap` 的分段锁）
5. ​**替代方案**​：在极端读多场景下，考虑 `StampedLock` 的乐观读模式
6. ​**避免过度同步**​：结合无锁数据结构（如 `Atomic` 类）减少锁竞争
#### 五、面试核心考点
1. ​**原理**​：AQS 实现、状态拆分、锁降级流程
2. ​**锁规则**​：读写互斥条件、公平性选择的影响
3. ​**使用场景**​：缓存系统、配置管理、数据库连接池等读多写少场景
4. ​**对比分析**​：与 `ReentrantLock`、`synchronized` 的差异
5. ​**问题排查**​：死锁、写饥饿的成因与解决方案

#### 六、适用场景
1. ​**缓存系统**​：高频读取缓存数据，低频更新（如商品信息查询）
2. ​**配置管理**​：多线程读取配置，偶发更新（如微服务配置中心）
3. ​**数据分析**​：实时统计数据的并发读取与批量更新
4. ​**文件/数据库操作**​：多线程读取文件内容，单线程写入日志
