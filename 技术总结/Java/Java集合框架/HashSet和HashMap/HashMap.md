---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - 集合
  - Java
  - HashMap
---
[[Java 哈希表]]
#### 一、底层数据结构
`HashMap` 的底层数据结构在不同` JDK` 版本中有所差异：
- `​JDK 7` 及之前**​：​**数组 + 链表**。通过哈希值计算桶（数组索引），冲突的元素以链表形式存储在同一个桶中。
- ​`JDK 8` 及之后**​：​**数组 + 链表 + 红黑树**。当链表长度超过阈值（默认 8）且数组长度 ≥ 64 时，链表会转换为红黑树（平衡二叉树），以降低查询时间复杂度（链表 `O(n)` → 红黑树 `O(logn)`）；当红黑树节点数 ≤ 6 时，会退化为链表（减少树结构的维护开销）。
#### 二、核心参数
`HashMap` 的构造函数支持自定义初始容量（`initialCapacity`）和负载因子（`loadFactor`），默认值为：
- ​**初始容量（Default Initial Capacity）​**​：`16`（数组的默认大小，必须是 2 的幂次）。
- ​**负载因子（Default Load Factor）​**​：`0.75f`（空间与时间的权衡：负载因子过小会导致频繁扩容，过大则链表/红黑树变长，查询变慢）。
- ​**扩容阈值（Threshold）​**​：`capacity * loadFactor`（当元素数量超过此值时触发扩容，默认 16 * 0.75=12）。
#### 三、关键操作流程
##### 1）put()方法流程
1. ​**计算哈希值**​：通过 `key.hashCode()` 得到原始哈希码，再通过 `(h = key.hashCode()) ^ (h >>> 16)` 高位参与运算，减少哈希冲突（将高位信息混合到低位）。
2. ​**确定桶位置**​：通过 `hash & (n - 1)`（`n` 为数组长度，且为 2 的幂次）计算桶索引（等价于 `hash % n`，但位运算更快）。
3. ​**插入或更新**​：
    - 若桶为空，直接创建新节点（`Node`）存入。
    - 若桶不为空，遍历链表/红黑树：
        - 若找到相同 `key`（`equals` 判断），覆盖旧值。
        - 若未找到，插入到链表尾部（`JDK 8` 改为尾插，避免 `JDK 7`头插导致的死循环）。
4. ​**树化（链表转红黑树）​**​：若链表长度 ≥ 8 且数组长度 ≥ 64，将链表转换为红黑树。
5. ​**扩容**​：若元素数量超过阈值（`size > threshold`），触发扩容（容量翻倍）。
##### 2)get()方法流程
1. 计算 `key` 的哈希值，确定桶位置。
2. 遍历桶中的链表或红黑树，通过 `equals` 匹配 `key`，返回对应值；若未找到，返回 `null`。
#### 四、扩容机制
- **触发条件**​：元素数量超过 `capacity * loadFactor`（默认 12）。
- ​**扩容方式**​：容量翻倍（如 16 → 32），创建新数组，重新哈希所有元素到新数组。
- ​**`JDK 7` 优化**​：扩容时通过 `e.hash & oldCap` 判断元素是新数组的 `i` 位置还是 `i + oldCap` 位置（减少哈希计算）。
- ​**`JDK 8` 优化**​：利用高位的额外信息（`(e.hash & oldCap) != 0`）直接决定新位置，避免重新计算哈希，提升效率。
#### 五、线程安全问题
`HashMap` ​**非线程安全**，多线程环境下可能出现以下问题：
- `​JDK 7`**​：多线程扩容时，链表可能因头插法形成环（死循环）。
- ​`JDK 8`**​：多线程插入时可能导致数据丢失（多个线程同时修改链表尾节点）。
- ​**解决方案**​：
    - 使用 `ConcurrentHashMap`（`JDK 7` 采用分段锁，`JDK 8` 采用 `CAS + synchronized`）。
    - 使用 `Collections.synchronizedMap(HashMap)`（外层加锁，性能较差）。
    - 业务层通过 `synchronized` 同步控制。
#### 六、`JDK7` vs`JDK8`核心差异
|​**特性**​|​**JDK 7**​|​**JDK 8**​|
|---|---|---|
|底层结构|数组 + 链表|数组 + 链表 + 红黑树|
|插入方式|头插法（可能导致死循环）|尾插法（避免死循环）|
|扩容优化|无|利用高位信息快速定位新桶位置|
|树化条件|无|链表长度 ≥ 8 且数组长度 ≥ 64|
|红黑树退化|无|红黑树节点数 ≤ 6 时退化为链表|
#### 七、常见面试题
1. ​**为什么 `HashMap` 的初始容量是 2 的幂次？​**​  
    为了通过位运算 `hash & (n - 1)` 快速计算桶索引（等价于取模），比 `%` 运算更高效；同时保证哈希值均匀分布，减少冲突。
    
2. ​**`HashMap` 允许 `key` 为 `null` 吗？如何处理？​**​  
    允许。`null` 的哈希值固定为 0，会被存入数组的 0 号桶（`table[0]`）。
    
3. ​**为什么负载因子默认是 0.75？​**​  
    是空间与时间的权衡：0.75 时哈希冲突概率较低，且内存利用率较高（超过 0.75 冲突增加，小于 0.75 则频繁扩容）。
    
4. ​**红黑树为什么能提高查询效率？​**​  
    红黑树是自平衡二叉树，插入、删除、查询的时间复杂度为 `O(logn)`，相比链表的 O(n) 更高效（尤其当链表较长时）。
    
5. ​**`HashMap` 的 `size` 为什么是 2 的幂次？如果传入非 2 幂次的值会怎样？​**​  
    构造函数中通过 `tableSizeFor(int cap)` 方法将任意正整数调整为大于等于 `cap` 的最小 2 的幂次（如传入 10 → 调整为 16），确保哈希计算的均匀性。
#### 八、`HashMap`使用
```java
System.out.println("----------------基础操作---------------------------");  
Map<String,Integer> map = new HashMap<>(); //定义一个Map  
map.put("key", 1); //插入键值对  
System.out.println("hashmap put key 1 "+ map);  
map.putIfAbsent("key", 2); //如果键不存时插入  
System.out.println("hashmap putIfAbsent key 2 "+ map);  
Integer i = map.get("key"); //获取值  
System.out.println("hashMap get key "+i);  
map.getOrDefault("key1", 2); //安全获取方式  
System.out.println("hashmap getOrDefault key1 defaultValue 2 "+map);  
boolean key1 = map.containsKey("key1"); //检查key存在性  
System.out.println("hashmap containsKey key1 "+ key1);  
map.compute("key", (k, v) -> v + 1);//原子性计算新值  
System.out.println("hashmap compute  " + map );  


System.out.println("---------------批量操作---------------------------");  
Map<String,Integer> newMap = new HashMap<>();  
newMap.put("key", 3); //有相同key不同值，被覆盖  
newMap.put("key1", 2);  
newMap.put("key2", 3);  
newMap.put("key3", 4);  
System.out.println("hashmap old map "+ map);  
map.putAll(newMap);  
System.out.println("hashmap putAll" + map);  
map.replaceAll((k,v) -> v + 1); //批量修改值  
System.out.println("hashmap replaceAll value+1 "+map);  


System.out.println("---------------视图方法---------------------------");  
Set<String> keys = map.keySet(); //获取键集合  
System.out.println("hashmap keys "+keys);  
Collection<Integer> values = map.values(); //值结合  
System.out.println("hashmap values "+values);  
Set<Map.Entry<String, Integer>> entries = map.entrySet(); //键值对集合  
System.out.println("hashmap entries "+entries);
```
##### 1）高级方法应用场景
###### ① 合并映射
使用 `merge()`方法实现键冲突处理
```java
map.merge("counter",1,Integer::sum); //存在则相加，存在则初始化为1
```
###### ② 缓存实现
利用 `computeifAbsent`实现延迟加载
```java
Map<String,Object> cache = new HashMap<>();
Object value = cache.computeifAbsent("key",k ->{
  return expensiveDatabaseQuery(k); //仅当不存在时执行查询
});
```
###### ③ 统计词频
结合流式API实现高效统计
```java
List<String> words = Arrays.asList("apple","banana","apple");
Map<String,Long> wordCount = words.stream()
.collect(Collectors.groupingBy(Function.identity(),Collectors.counting()));
//分组统计
```
###### ④ 遍历优化
优先使用`entrySet`迭代（比先取`keySet`再get效率高20%）：
```java
for(Map.Entry<String,Integer> entry:map.entrySet()){
   System.out.println(entry.getKey()+":"+entry.getValue())
}
```
#### 九、哈希冲突处理与扩容机制
##### 1）冲突解决：
- ​**链地址法**：冲突元素以链表存储，Java 8 后引入红黑树优化长链表
##### 2）​**动态扩容**：
- ​**触发条件**：当元素数量超过 `容量 × 负载因子（默认 0.75）` 时触发
- **扩容过程**：新容量为原 2 倍，重新哈希所有元素到新数组，逐步迁移以减少性能损耗
#### 十、线程安全与并发问题
##### 1）非线程安全：
多线程并发修改可能导致数据不一致、循环链表（`JDK 1.7` 前）等问题
##### 2）​**解决方案**：
###### ① ​[[ConcurrentHashMap]]
基于分段锁（`JDK 1.7`）或 `CAS` + 同步块（`JDK 1.8`）实现高并发安全
```java
//并发容器
ConcurrentHashMap<String,Integer> concurrentMap = new ConcurrentHashMap<>();
```
###### ② 包装类`synchronizedMap`
通过同步包装类实现线程安全，但性能较低
```java
//同步包装类
Map<String,Integer> syncMap = Collections.synchronizedMap(new HashMap<>());
```
###### ③ 读写分离
`computeIfAbsent` 实现读写分离方案

```java
//读写分离方案
Map<String,LongAdder> counterMap = new HashMap<>();
LongAdder adder = counterMap.computeifAbsent("key",k -> new LongAdder());
adder.increment();
```
#### 十一、性能优化与使用建议
##### 1）初始化参数
- 初始容量：预估元素数量，避免频繁扩容（建议 `初始容量 = 预期元素数 / 负载因子 + 1`）
- ​负载因子：默认 0.75 平衡时间与空间效率

```java
//预估存储100个元素时的初始化方式
int initialCapacity = (int)(100/0.75)+1; //避免频繁扩容
Map<String,Object> optimizedMap = new HashMap<>(initialCapacity);
```

##### 2）**键设计**：
- ​**不可变对象**：如 `String`、`Integer`，避免修改 Key 导致哈希值变化
- **重写 `hashCode()` 和 `equals()`：**确保哈希分布均匀且相等性判断正确
- 重写`hashCode()`时使用`Objects.hash()`自动组合字段

##### 3）调试
###### ① 哈希碰撞检测
```java
//通过监测链表长度发现潜在问题
System.out.println("TREEIFY_THRESHOLD:" + HashMap.TREEIFY_THRESHOLD); //默认8
```
###### ② 内存分析
使用`JVisualVM`查看`HashMap`实例的
- Table数组大小
- 加载因子分布
- 链表/红黑树比例
#### 十二、常见问题与注意事项
###### ① 哈希冲突过多
可能因 Key 的哈希函数设计不佳导致性能下降，需优化哈希算法
###### ② 内存泄漏：
长期持有 `HashMap` 引用导致无法回收，需及时清理无用键值对