---
创建时间: 2025-04-10 12:00:57
作者: wangxiaoming
tags:
  - 数据结构
  - 哈希表
  - Java
  - HashMap
  - HashSet
---
[[常见数据结构#1）哈希表（Hash Table）]]
#### 一、哈希表核心结构
##### 1）底层实现演变
- **JDK7及之前：**数据+链表结构，没个链表称为一个桶(bucket)
- **JDK8及之后：**数组+链表+红黑树，当链表长度>=8且长度>=64时，链表自动转为红黑树
![[Pasted image 20250410211304.png]]
##### 2）关键参数
- **初始容量：**默认16（建议预估数据量设置初始值，会被自动对齐到最近的2的幂）
- **加载因子：**默认0.75,当已用容量达到总容量*0.75时触发扩容（数组翻倍）
- **扰动函数：**通过`(h = key.hashCode()^(h>>>16))`

#### 二、对象处理机制
##### 1）哈希值计算
```java
//定位桶位置的核心算法（n为数组长度）
i = (n - 1) & hash //等价于取模运算，效率更高
```
##### 2）重复判断逻辑
- 先比价哈希值：不同则直接存入
- 哈希值相同则调用`equals()`方法进行深度比较
- 必须同时重写`hashCode()` 和 `equals()` 才能正确去重（如String类已实现）

#### 三、核心使用技巧
##### 1）HashSet特性 [[HashSet]]
```java
Set<String> set = new HashSet<>();
set.add("A"); //实际存储为HashMap的Key
```
- 不允许重复元素（依赖`hashCode`+`equals`判断）
- 线程不安全，允许`null`值
##### 2）HashMap特性 [[HashMap]]
```java
Map<String,Integer> map =new HashMap<>();
map.put("key",1);
map.getOrDefault("key",0);
```
- 键唯一，值可重复
- 允许一个null键和多个null值
-  ​**无序性**：不保证插入顺序，需有序存储可使用 `LinkedHashMap`
- ​**高效操作**：平均时间复杂度为 O(1)，查找、插入、删除性能优异

#### 四、性能优化要点
##### 1）避免哈希碰撞
- 自定义对象需合理设计`hashCode()`，推荐使用 `Object.hash(field1,field2)`
- 示例：两个不同Student对象哈希值相同会导致哈希碰撞
##### 2）扩容策略
- 初始容量建议设置为预估元素数量/0.75 + 1 (避免频繁扩容)
- 扩容时所有元素需要重新计算哈希位置（耗时操作）

#### 五、源码关键设计
##### 1）tableSizeFor方法
```java
//将任意数值转换为2的幂（如输入18->32）
static final int tableSizeFor(int cap){
   int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
   return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
##### 2）红黑树转换阈值
链表长度>=8且数组长度>=64时转换红黑树（查询时间复杂度从O(n) ->O(logn)）