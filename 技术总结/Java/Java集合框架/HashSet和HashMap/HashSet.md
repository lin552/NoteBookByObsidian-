---
创建时间: 2025-04-11 18:12:22
作者: wangxiaoming
tags:
  - Java
  - HashSet
---
[[HashMap]]
#### 一、HashSet基于HashMap的核心实现机制
`HashSet`是java集合框架中基于`HashMap`实现的轻量化集合类，其设计理念是通过复用`HashMap`的键唯一特性来实现元素去重

##### 1）键值映射的巧妙简化
存储结构：`HashSet` 内部维护一个 `HashMap<E,Object>`实例，所有元素作为`HashMap`的键(Key),而值(Value)同一使用静态常量`PRESENT` （一个空的Object对象）。
去重原理：当调用`add(E e)`方法时，实际执行`map.put(e,PRESENT)`。若键已存在，返回`false`；若不存在，返回`true`。这一机制直接继承了`HashMap`的键唯一性特性。

##### 2）构造函数的灵活适配
默认构造：直接利用`HashMap`的链表+红黑树结构。当链表长度>=8且数据容量>=64时，链表自动转为红黑树，将查询复杂度从O(n)降为O(logn)。
扩容机制：与`hashMap`一致，当元素数量超过 `容量 x 容量因子`时，触发翻倍扩容并重新哈希分配元素。

#### 二、HashSet的用法和注意事项
##### 1）用法
```java
System.out.println("----------------基础操作---------------------------");  
Set<String> set = new HashSet<>();  
set.add("A"); //添加元素（自动去重）  
System.out.println("hashset put key A "+ set);  
set.remove("B"); //删除元素  
System.out.println("hashset remove key B "+ set);  
boolean c = set.contains("C");//存在性检测  
System.out.println("hashset containsKey keyC "+ c);  
Set<String> newSet = new HashSet<>();  
newSet.add("A");  
newSet.add("B");  
newSet.add("C");  
newSet.add("D");  
set.add("C");  
set.add("B");  
System.out.println("hashset old set "+ set);  
System.out.println("hashset new set "+ newSet);  
boolean b = set.retainAll(newSet);//求交集 返回值  
System.out.println("hashset retainAll求交集 newSet "+ set+" boolean "+b);  
List<String> rawData = Arrays.asList("A","B","C","D","D","F");  
Set<String> uniqueSet = new HashSet<>(rawData); //自动去重  
System.out.println("hashSet 批量自动去重List list"+rawData+" uniqueSet "+uniqueSet);  
  
System.out.println("----------------集合运算示例-----------------------");  
Set<String> union = new HashSet<>(set);  
union.addAll(newSet); //并集操作  
System.out.println("hashset addAll并集操作 "+ union);
```

##### 2）使用注意事项
###### ① 线程不安全
多线程环境下需使用`Collections.synchronizedSet(new HashSet<>())`
```java
Set<String> stringSet = Collections.synchronizedSet(new HashSet<String>());//使用synchronizedSet  
stringSet.add("Set多线程添加1");  
stringSet.add("Set多线程添加2");  
System.out.println("hashSet 多线程 synchroinizedSet objects "+stringSet);
```
或改用 `ConcurrentHashMap.KeySetView
```java
//使用ConcurrentHashMap.newKeySet();  
Set<String> keySet = ConcurrentHashMap.newKeySet(); 
keySet.add("Set多线程newKeySet添加key2");  
System.out.println("hashset 多线程ConcurrentHashMap newKeySet "+keySet);  
  
ConcurrentHashMap<String,Integer> map = new ConcurrentHashMap<>();  
map.put("ConcurrentHashMap Key1",1);  
map.put("ConcurrentHashMap Key2",2);  
//这种方法返回的集合与原始 ConcurrentHashMap 动态关联，修改映射会实时反映到集合中  
ConcurrentHashMap.KeySetView<String, Integer> keySet = map.keySet(); //返回keySetView  
System.out.println("Set ConcurrentHashMap keySet "+keySet);  
keySet.remove("ConcurrentHashMap Key1");  
System.out.println("Set ConcurrentHashMap keySetView Remove key1 原始Map "+map);  
//strings.add("ConcurrentHashMap Key3"); //无法add会报错 

//带默认值的KeySetView  
UnsupportedOperationExceptionConcurrentHashMap map2 = new ConcurrentHashMap(); 
ConcurrentHashMap.KeySetView<String, String> keySet2 = map2.keySet("DEFAULT_VALUE");
keySet2.add("key2"); //等同于put(key2,"DEFAULT_VALUE");  
System.out.println("Set ConcurrentHashMap keySetDefaultValue put key2 "+ map2);
```
###### ② 自定义对象处理
必须重写`hashCode()`和 `equals()`方法，否则可能导致重复元素未被正确过滤
```java
@Override
public int hashCode(){
   return Objects.hash(name,age); //组合关键字段生成哈希值
}
```
###### ③ 遍历时修改风险
迭代过程中修改集合会触发`ConcurrentModificationException`,需通过迭代器 remove()方法操作

**快速失败(Fail-Fast)机制：**
`HashSet`的迭代器在遍历时会检查集合的修改次数(`modCount`)。如果遍历过程中集合被修改（非迭代器自身操作），会立即抛出`ConcurrentModifcationException`
**弱一致性(Weakly Consistent)迭代器：**
并发集合类（如 `ConcurrentHashMap.KeySetView`）的迭代器是弱一致性的，允许在遍历时修改集合，但可能不会实时反映所有修改，避免抛出异常
```java
System.out.println("----------------HashSet安全遍历-----------------------");  
Set<String> mySet = new HashSet<>(Arrays.asList("A","B","C"));  
Iterator<String> setIterator = mySet.iterator();  
System.out.println("Set 使用遍历器安全 remove 前 "+mySet);  
while(setIterator.hasNext()){  
    String next = setIterator.next();  
    if (next.equals("A")){  
        setIterator.remove(); //移除内容  
        System.out.println("Set 使用遍历器安全 remove 后 "+mySet);  
    }  
}
```
###### ④ 空值限制
允许单个`null`元素，但需注意业务场景中`null`可能引起的歧义

#### 三、性能调优技巧
##### 1）初始化参数优化
- ​**初始容量**​：根据预估元素数量计算，公式为 `(预估数量 / 0.75) + 1`。例如，预计存储 1000 个元素，初始容量应设为 `(1000 / 0.75) + 1 ≈ 1334`
- ​**负载因子**​：默认 0.75 是空间与时间的平衡点。若内存充足且追求极致查询性能，可降低至 0.5；若内存紧张，可适当提高至 0.9，但会增加哈希冲突概率
```java
System.out.println("----------------HashSet调休技巧-----------------------");  
//预期存储1000个元素，负载因子0.75  
int expectedSize = 1000;  
float loadFactor=0.75f;  
int initialCapacity = (int)(expectedSize/loadFactor)+1;  
//initialCapacity 会调整为2的幂 （如设置1000会调整为1024）  
HashSet<String> optimizedSet = new HashSet<>(initialCapacity,loadFactor);  
//插入数据  
for (int i = 0; i < expectedSize; i++) {  
    optimizedSet.add("Item-"+i);  
}  
System.out.println("Set 初始化容量 optimizedSet "+optimizedSet);  
  
//通过反射查看实际容量(仅调试用)  
try{  
    Field field = Set.class.getDeclaredField("optimizedSet");  
    field.setAccessible(true);  
    HashMap<?,?> internalMap = (HashMap<?,?>)field.get(optimizedSet);  
    System.out.println("Set 反射查实际容量 internalMap "+internalMap); //输出桶的数量  
} catch (Exception e){  
    e.printStackTrace();  
}
```
##### 2）哈希冲突优化
​**均匀哈希算法**​：自定义对象的 `hashCode()` 应尽量分散，避免大量元素落入同一哈希桶。例如，对字符串哈希值进行扰动：
```java
public int hashCode(){
  int h = key.hashCode();
  return h ^ (h >>> 16); //高低位异或减少碰撞
}
```
​**避免复杂对象**​：若元素为复杂对象（如嵌套集合），需确保哈希计算不会频繁变化，否则会导致哈希表频繁重构
##### 3）场景适配与替代方案
有序需求：改用 `LinkedHashSet` (基于链表维护插入顺序)或`TreeSet` (基于红黑树实现排序)
搞并发场景：优先选择 `ConcurrentHashMap.newKeySet()` 替代 `Collecations.synchronizedSet()`，前者采用分段锁，并发性能更优。