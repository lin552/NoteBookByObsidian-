---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - 集合
  - Java
  - HashMap
---
[[Java 哈希表]]
#### 一、HashMap使用
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
优先使用`entrySet`迭代（比先取keySet再get效率高20%）：
```java
for(Map.Entry<String,Integer> entry:map.entrySet()){
   System.out.println(entry.getKey()+":"+entry.getValue())
}
```
#### 二、哈希冲突处理与扩容机制
##### 1）冲突解决：
- ​**链地址法**：冲突元素以链表存储，Java 8 后引入红黑树优化长链表
##### 2）​**动态扩容**：
- ​**触发条件**：当元素数量超过 `容量 × 负载因子（默认 0.75）` 时触发
- **扩容过程**：新容量为原 2 倍，重新哈希所有元素到新数组，逐步迁移以减少性能损耗
#### 二、线程安全与并发问题
##### 1）非线程安全：
多线程并发修改可能导致数据不一致、循环链表（JDK 1.7 前）等问题
##### 2）​**解决方案**：
###### ① ​[[ConcurrentHashMap]]
基于分段锁（JDK 1.7）或 CAS + 同步块（JDK 1.8）实现高并发安全
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
#### 三、性能优化与使用建议
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
使用JVisualVM查看HashMap实例的
- Table数组大小
- 加载因子分布
- 链表/红黑树比例
#### 四、常见问题与注意事项
###### ① 哈希冲突过多
可能因 Key 的哈希函数设计不佳导致性能下降，需优化哈希算法
###### ② 内存泄漏：
长期持有 HashMap 引用导致无法回收，需及时清理无用键值对