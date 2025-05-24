---
创建时间: 2025-04-11 22:11:34
作者: wangxiaoming
tags:
  - LinkedHashMap
  - LinkedHashSet
---
#### 一、基本原理
##### 1）`LinkedHashMap`
- 继承关系：继承自`HashMap`,底层基于哈希表+双向链表实现
###### ① 特性：
- 维护顺序：默认按插入顺序排序，通过双向链表记录元素顺序，若设置`accessOrder = true` 则按访问顺序排序（适合实现`LRU`缓存）
- 性能优化：迭代时直接遍历链表，时间复杂度为O(n)，与哈希表容量无关。
###### ② 关键方法：
- 重写`newNode()`方法，在插入时通过`linkNodeList()`将节点链入双向链表尾部
- `removeEldestEntry()`方法用于实现缓存淘汰策略（如`LRU`）

##### 2)`LinkedHashSet`
- 继承关系：继承自`HashSet`，底层基于`LinkedHashMap`实现，仅存储键（值统一为`PRESENT`占位对象）
###### ① 和`LinkedHashMap`关系：
**实现依赖：**`LinkedHashSet`是 `LinkedHashMap` 的适配器模式实现，其构造方法内部初始化 `LinkedHashMap`
**功能差异：**
- `LinkedHashMap`存储键值对，`LinkedHashSet` 仅储存唯一元素
- 两者均通过双向链表维护顺序，但应用场景不同

###### 特性：
- 元素唯一且按插入顺序迭代，本质是`LinkedHashMap`的键集合
- 性能与`LinkedHashMap`一致，添加/删除/查找时间复杂度为O(1)

#### 二、基础使用
##### 1）`LinkedHashMap`使用
###### 基本使用
```java
System.out.println("-------------LinkedHashMap使用----------------------------------");  
LinkedHashMap<String,Integer> linkedHashMap = new LinkedHashMap<>();  
linkedHashMap.put("Apple", 100);  
linkedHashMap.put("Banana", 200);  
linkedHashMap.put("Orange", 300);  
Integer i = linkedHashMap.get("Apple");  
System.out.println("LinkedHashMap get apple "+i);  
Integer apple = linkedHashMap.remove("Apple");  
System.out.println("LinkedHashMap remove apple "+apple);  
boolean b = linkedHashMap.containsKey("Banana");  
System.out.println("LinkedHashMap contains banana "+b);  
System.out.println("LinkedHashMap now map "+linkedHashMap);  
  
linkedHashMap.forEach((k,v)->{  
    System.out.println("LinkedHashMap 遍历 key "+k+" value "+v);  
});

System.out.println("-------------LinkedHashSet线程安全优化----------------------------------");  
Map<String,Integer> safeMap = Collections.synchronizedMap(new LinkedHashMap<>());
```
###### 访问顺序模式（`LRU基础`）
```java
System.out.println("-------------LinkedHashMap LRU基础----------------------------------");  
LinkedHashMap<String,Integer> lruCache = new LinkedHashMap<>(16,0.75f,true){  
    protected boolean removeEldestEntry(Map.Entry<String, Integer> eldest) {  
        return size() > 3;  
    }  
};  
lruCache.put("A", 100);  
System.out.println("LinkedHashMap LRU lruCache add A"+lruCache);  
lruCache.put("B", 200);  
System.out.println("LinkedHashMap LRU lruCache add B"+lruCache);  
lruCache.put("C", 300);  
System.out.println("LinkedHashMap LRU lruCache add C"+lruCache);  
lruCache.put("A", 100); //触发访问顺序调整  
System.out.println("LinkedHashMap LRU lruCache 2add A"+lruCache);  
lruCache.put("D", 400);  
System.out.println("LinkedHashMap LRU lruCache add D"+lruCache);  
lruCache.put("E", 500); //淘汰最老的元素“B”  
System.out.println("LinkedHashMap LRU lruCache add E"+lruCache);  
System.out.println("LinkedHashMap LRU lruCache"+lruCache);
```

##### 2）`LinkedHashSet` 使用
```java
System.out.println("-------------LinkedHashSet 使用----------------------------------");  
LinkedHashSet<String> linkedHashSet = new LinkedHashSet<>();  
linkedHashSet.add("Apple");  
linkedHashSet.add("Banana");  
linkedHashSet.add("Orange");  
  
for(String fruit : linkedHashSet) {  
    System.out.println("LinkedHashSet 遍历Set "+fruit);  
}

System.out.println("-------------LinkedHashSet线程安全优化----------------------------------");  
Set<String> safeSet = Collections.synchronizedSet(new LinkedHashSet<>());
```
记录访问轨迹
```java
System.out.println("-------------LinkedHashSet记录访问轨迹----------------------------------");  
LinkedHashSet<String> visitHistory = new LinkedHashSet<>();  
visitHistory.add("/home");  
visitHistory.add("/products");  
visitHistory.add("/contact");  
visitHistory.add("/home"); //重复访问被忽略  
System.out.println("LinkedHashSet 记录访问轨迹 visitHistory "+visitHistory);
```

#### 三、使用与优化建议
##### 1) 使用场景
###### `LinkedHashMap`
- 需要维护插入或访问顺序的缓存（如LRU）
- 需要按顺序遍历键值对的场景（如日志记录）
###### `LinkedHashSet`
- 需要去重保持插入顺序的集合（如用户访问历史记录）
- 替代`HashSet`以提升迭代性能
##### 2）优化建议
- 初始容量与负载因子：预设合理容量（如预估元素的1.5倍），减少扩容开销
- 线程安全：默认非线程安全，多线程环境下需通过`Collections.synchronizedMap/Set`包装
- `LRU`实现：通过`accessOrder = true`和重写`removeEldestEntry()`实现缓存淘汰