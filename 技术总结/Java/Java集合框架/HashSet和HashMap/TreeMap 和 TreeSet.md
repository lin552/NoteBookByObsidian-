---
创建时间: 2025-04-11 21:28:58
作者: wangxiaoming
tags:
  - TreeMap
  - TreeSet
---
#### 一、核心原理
##### 1）`TreeMap`
- **数据结构**：基于`红黑树（Red-Black Tree）`实现的自平衡二叉查找树
- **有序性**：**键(Key)按自然顺序(Comparable)或自定义比较器(`Comparator`)排序
- **性能**：**插入、删除、查询操作的时间复杂度为O(log n)
- **线程安全**：**非线程安全，需外部同步或使用并发集合。
##### 2）`TreeSet`
- **数据结构**：基于`TreeMap`实现,元素作为`TreeMap`的键，值统一为固定占位对象（如 PRESENT）
- **有序性**：元素唯一且按自然顺序或自定义比较器排序
- **性能**：操作时间复杂度同 `TreeMap` (O(log n))
- **线程安全**：非线程安全
#### 二、`TreeSet`对`TreeMap`的封装与改进
###### 封装关系
`TreeSet` 内部持有一个`TreeMap`实例，所有操作（如 `add`、`remove`）均委托给`TreeMap`
```java
public boolean add(E e){
   return map.put(e,PRESENT) == null; //调用TreeMap的put方法
}
```
###### 改进点
`TreeSet` 通过封装 `TreeMap` 实现了 `Set`接口，将键值对的键作为集合元素，值无意义，从而简化了有序唯一集合的实现。

#### 三、使用方式
##### 1）`TreeMap`使用
###### ① 自然排序
```java
System.out.println("---------TreeMap使用---------------");  
TreeMap<String,Integer> treeMap = new TreeMap<>();  
treeMap.put("Apple", 1);  
treeMap.put("Cat", 3);  
treeMap.put("Banana", 2);  
System.out.println("treeMap 自然排序 "+treeMap);
```
###### ② 自定义排序 
```java
System.out.println("---------TreeMap自定义排序---------------");  
TreeMap<String,Integer> customTreeMap = new TreeMap<>(  
        (a,b) ->b.compareTo(a) //降序排序  
);  
customTreeMap.put("A", 1);  
customTreeMap.put("C", 3);  
customTreeMap.put("B", 2);  
System.out.println("treeMap 自定义排序降序 "+customTreeMap);

//TreeSet中提供了众多的实现Comparator的方法，在此不重复
```
###### ③ 范围查询
```java
System.out.println("---------TreeMap范围查询---------------");  
TreeMap<String,Integer> sorted = new TreeMap<>();  
sorted.put("AAA",1);  
sorted.put("BBB",2);  
sorted.put("CCC",3);  
sorted.put("DDD",4);  
sorted.put("EEE",5);  
//按照范围查询 boolean表示是否包含首尾  
SortedMap<String,Integer> subMap = sorted.subMap("BBB", true, "DDD", true);  
System.out.println("TreeMap 范围查询 BBB 到 DDD "+subMap);
```

##### 2）`TreeSet`使用
###### ① 自然排序
```java
System.out.println("---------TreeSet使用---------------");  
TreeSet<String> treeSet = new TreeSet<>();  
treeSet.add("Apple");  
treeSet.add("Cat");  
treeSet.add("Banana");  
treeSet.add("apple");  
treeSet.add("banana");  
treeSet.add("111");  
treeSet.add("222");  
System.out.println("TreeSet 自然排序使用 "+treeSet);
```
###### ② 自定义排序
```java
System.out.println("---------TreeSet 自定义反序------------");  
TreeSet<String> customTreeSet = new TreeSet<>(Comparator.reverseOrder());  
customTreeSet.add("Apple");  
customTreeSet.add("Banana");  
System.out.println("TreeSet 自定义排序 "+customTreeSet);

System.out.println("---------TreeSet 基于对象属性------------");  
//适用场景:按对象的某个属性（如年龄、价格）排序  
TreeSet<Student> students1 = new TreeSet<>(  
        Comparator.comparingInt(Student::getAge)  
);  
students1.add(new Student("Alice",20));  
students1.add(new Student("Bob",15));  
System.out.println("TreeSet 基于对象属性对比 studentSet " + students1);  
  
  
System.out.println("---------TreeSet匿名内部类------------");  
//适用场景：需要复杂比较逻辑  
TreeSet<Student> students2 = new TreeSet<>(new Comparator<Student>() {  
    @Override  
    public int compare(Student o1, Student o2) {  
        return Integer.compare(o1.getAge(), o2.getAge());  
    }  
});  
students2.add(new Student("小明",30));  
students2.add(new Student("小美",11));  
System.out.println("TreeSet 匿名内部类实现对比 students "+students2);  
  
System.out.println("---------TreeSet实现Comparator接口------------");  
TreeSet<Student> students3 = new TreeSet<>(new StudentComparator());  
students3.add(new Student("Alice",20));  
students3.add(new Student("Bob",15));  
students3.add(new Student("Mike",30));  
System.out.println("TreeSet 实现Comparator接口 students "+students3);  
  
System.out.println("---------TreeSet 链式比较器（多级排序）------------");  
//适用场景：需要多个排序条件的组合（如先按部门排序，再按工号排序）  
Comparator<Student> comparator = Comparator  
        .comparing(Student::getAge) //第一级 年龄  
        .thenComparing(Student::getName); //第二级 姓名  
  
TreeSet<Student> students4 = new TreeSet<>(comparator);  
students4.add(new Student("Alice",20));  
students4.add(new Student("Bob",15));  
students4.add(new Student("Mike",20));  
System.out.println("TreeSet 链式比较器 students "+students4);  
  
System.out.println("---------TreeSet 处理null值------------");  
//适用场景:集合中允许包含 null 元素并定义其排序位置  
//null 视为最小值，排在前面  
Comparator<String> nullsFirstComparator = Comparator.nullsFirst(Comparator.naturalOrder());  
TreeSet<String> nullsFirstTreeSet = new TreeSet<>(nullsFirstComparator);  
nullsFirstTreeSet.add("Alice");  
nullsFirstTreeSet.add(null);  
System.out.println("TreeSet 处理null的比较器 nullsFirstComparator "+nullsFirstTreeSet);  
  
System.out.println("---------TreeSet 自定义规则（非自然属性）------------");  
//适用场景：按非自然属性排序（如字符串的倒数第三个字符）  
TreeSet<String> diySet  = new TreeSet<>(  
        (a,b) ->{  
            char c1 = a.charAt(a.length()-3);  
            char c2 = b.charAt(b.length()-3);  
            return Character.compare(c1,c2);  
        }  
);  
diySet.add("Alice");  
diySet.add("Bob");  
diySet.add("Mke");  
diySet.add("Jack");  
diySet.add("Kim");  
System.out.println("TreeSet 自定义规则非自然属性 diySet "+diySet);
```
以上用到的类 Student和 StudentComparator
```java
/**  
 * 自定义学生比较类  
 */  
static class StudentComparator implements Comparator<Student> {  
  
    @Override  
    public int compare(Student o1, Student o2) {  
        return Integer.compare(o1.getAge(), o2.getAge());  
    }  
}  
  
/**  
 * 学生实体  
 */  
static class Student {  
    String name;  
    int age;  
  
    public Student(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
  
    public int getAge() {  
        return age;  
    }  
  
    public String getName() {  
        return name;  
    }  
  
    @Override  
    public String toString() {  
        return  "Student{name='" + name + "', age=" + age + "}";  
    }  
  
    @Override  
    public boolean equals(Object obj) {  
        if(this == obj) return true;  
        if(obj == null || getClass() != obj.getClass()) return false;  
        Student student = (Student) obj;  
        return age == student.age && Objects.equals(name, student.name);  
    }  
  
    @Override  
    public int hashCode() {  
        return Objects.hash(name, age);  
    }  
}
```
###### ③ 范围操作
```java
System.out.println("---------TreeSet 范围查找------------");  
TreeSet<String> sortedTreeSet = new TreeSet<>();  
sortedTreeSet.add("AAA");  
sortedTreeSet.add("BBB");  
sortedTreeSet.add("CCC");  
sortedTreeSet.add("DDD");  
NavigableSet<String> subSet = sortedTreeSet.subSet("BBB", true, "DDD", true);  
System.out.println("TreeSet 范围查找 BBB 到 DDD "+subSet);
```
#### 四、使用注意事项
###### 1.元素必须可比较
元素需实现`Comparable`接口，或在构造时提供`Comparator`,否则抛出`ClassCastException`
###### 2.线程安全
线程非安全，多线程环境需同步或改用并发集合(如`ConcurrentSkipListSet`)
###### 3.迭代器快速失败
迭代过程中若结构被修改，会抛出`ConcurrentModificationException`
###### 4.性能权衡
插入/删除效率低于`HashSet`（O(1) vs O(log n)），适合需要有序性的场景

#### 五、性能优化建议
###### 1.高效比较器
自定义`Comparator` 应避免复杂计算，以提升比较速度
###### 2.避免频繁修改
红黑树的平衡操作有开销，频繁插入/删除时考虑`HashSet` + 手动排序
###### 3.范围查询优化
利用`subMap()`、`heapMap`、`tailMap` 进行批量操作，减少遍历次数

#### 六、适用场景
##### 1）`TreeSet
###### 适用场景
- 需要元素有序的唯一集合（如排行榜、字典序存储）
- 需要频繁的范围查询（如统计某个区间内的数据）
- 需要快速获取最大/最小元素（如优先队列功能）
###### 不适用场景
- 高频插入/删除且无需排序（性能低于`HashSet`）
- 内存敏感场景（红黑树结构占用更多内存）
##### 2）`TreeMap
###### 适用场景
- 需要键有序的键值对存储（如配置项按字母排序）
- 需要频繁的范围查询（如时间区间内的日志统计）
- 需要快速获取最大/最下键（如极值统计）
###### 不适合场景
- 高频插入/删除且无需排序（性能低于`HashMap`）
- 内存敏感场景（红黑树节点占用更多内存）