---
创建时间: 2025-04-11 22:10:56
作者: wangxiaoming
tags:
  - ConcurrentHashMap
  - ConcurrentSkipListSet
---
#### 一、原理与数据结构
##### 1）跳表（`SkipList`）的核心原理
- **数据结构设计**：跳表通过多层有序链表实现，每一层是下一层的子集。底层链表包含所有元素，上层链表作为索引层，加速查找
- **查找与插入**：查找时从最高层索引开始，逐层向下跳跃，时间复杂度为O（log n）；插入时通过随机生成层数（概率决定）更新索引结构
- **线程安全机制**：采用无锁编程（`CAS`操作）和细粒度锁分段技术，避免全局锁竞争，支持高并发读写
##### 2）`ConcurrentSkipListMap`的实现
- **存储结构**：每个节点（`Node`）包含键值对，索引节点（`Index`）维护多层链表的引用（`right` 和 `down`指针）
- **有序性**：默认按键的自然顺序或自定义比较器排序，支持范围查询（如 `subMap`、`headMap`）和导航操作（如 `ceilingKey`）
##### 3）`ConcurrentSkipListSet` 的实现
- 基于 `ConcurrentSkipListMap`，将元素作为键存储，值为固定占位对象（如 `Boolean.TRUE`），复用跳表的有序性和并发控制机制

#### 二、基础用法
##### 1）`ConcurrentSkipListMap`
```java
System.out.println("--------------------ConcurrentSkipListMap使用---------------------");  
ConcurrentSkipListMap<Integer,String> map = new ConcurrentSkipListMap<>();  
map.put(5, "five");  
map.put(3, "three"); //插入元素（自动排序）  
map.put(4, "four");  
System.out.println("ConcurrentSkipListMap put 3");  
String s = map.get(3);//查找值  
System.out.println("ConcurrentSkipListMap get 3 "+s);  
String remove = map.remove(3);  
System.out.println("ConcurrentSkipListMap remove 3 "+remove);  
map.entrySet().forEach(System.out::println); //有序遍历  
map.computeIfAbsent(5, k -> "default");//原子性条件插入  
System.out.println("ConcurrentSkipListMap computeIfAbsent 5 "+map);
```
##### 2）`ConcurrentSkipListSet`
```java
System.out.println("--------------------ConcurrentSkipListSet使用---------------------");  
ConcurrentSkipListSet<Integer> set = new ConcurrentSkipListSet<>();  
set.add(3);  
set.add(5);  
set.add(4);  
set.add(2);  
set.add(1);  
boolean five = set.contains(5);  
System.out.println("ConcurrentSkipListSet contains five "+five);  
int first = set.first();  
System.out.println("ConcurrentSkipListSet get first 最小元素 "+first);  
NavigableSet<Integer> integers = set.tailSet(2);//获取大于等于2的子集  
System.out.println("ConcurrentSkipList set tail 2 "+integers);
```

#### 三、性能优化建议
##### 1）适用场景选择
- **优先使用场景**：需有序且高并发（如范围查询，排序遍历）时选择跳表结构；若仅需哈希表的高吞吐量，则用`ConcurrentHashMap`
- **避免场景**：单线程或低并发时，优先用`TreeMap`或`TreeSet`以减少内存开销
##### 2）内存与计算优化
- **减少`size()`调用**：`size()`需遍历链表，时间复杂度为O(n)，建议通过计数器维护大小
- **合理设置比较器**：自定义比较器应高效，避免复杂计算影响排序性能

##### 3）并发调优
- **批量操作限制**：`addAll`、`removeAll` 等批量方法非原子性，需外部同步
- **读多写少优化**：跳表在频繁查询时性能优异，写入频繁时可考虑分段合并策略