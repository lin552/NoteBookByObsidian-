---
创建时间: 2025-04-12 17:24:57
作者: wangxiaoming
tags:
  - ArryList
  - LinkedList
  - Java
  - 集合
---
##### 一、底层数据结构
###### `ArrayList`
- 实现：基于**动态数组**实现，元素在内存中连续存储
- 检索方式：支持快速访问（通过**索引**直接定位元素）
- 扩容机制：默认容量10，扩容时容量增加50%(10 -> 15 -> 22)
###### `LinkedList`
- 基于双向链表实现，每个节点包含数据及其前后指针
- 检索方式：元素在内存中非连续存储，可动态增删
- 支持操作：支持堆栈（LIFO）和队列（FIFO）操作（ 如 addFirst()、pollLast() ）
##### 二、性能对比
| ​**操作类型**​         | ​**ArrayList 时间复杂度**​ | ​**LinkedList 时间复杂度**​ | ​**实际场景性能对比（10万数据量）​**​                   |
| ------------------ | --------------------- | ---------------------- | ----------------------------------------- |
| ​**随机访问（get）​**​   | O(1)                  | O(n)                   | ArrayList 快 1000+ 倍（3ms vs 403ms）<br>     |
| ​**尾部插入（add）​**​   | O(1)（摊还）              | O(1)                   | 性能接近（26ms vs 28ms）<br>                    |
| ​**头部插入（add）​**​   | O(n)                  | O(1)                   | LinkedList 快 57 倍（15ms vs 859ms）<br>      |
| ​**中间插入（add）​**​   | O(n)                  | O(n)（需遍历到位置）           | LinkedList 慢 8.6 倍（15981ms vs 1848ms）<br> |
| ​**尾部删除（remove）​** | O(1)                  | O(1)                   | 性能接近                                      |
| ​**头部删除（remove）​** | O(n)                  | O(1)                   | LinkedList 快 50+ 倍                        |
| ​**中间删除（remove）​** | O(n)                  | O(n)（需遍历到位置）           | LinkedList 更慢（实测差异显著）<br>                 |
| ​**内存占用**​         | 紧凑（仅元素存储）             | 松散（每个节点+2指针）           | LinkedList 多消耗 3-7 倍内存<br><br>            |
| ​**缓存友好性**​        | 高（连续内存预读）             | 低（内存碎片化）               | ArrayList 遍历快 2-5 倍<br><br>               |

##### 三、内存占用
- `ArrayList`:仅存储数据，内存开销较小
- `LinkedList`:每个节点需额外存储前后指针，内存占用较高
##### 四、适用场景示例
###### 1.使用`ArrayList`场景
- 存储用户信息需快速查询
- 数据量较小且操作查询为主
###### 2.使用`LinkedList`场景
- 频繁插入/删除场景 例如聊天消息列表
- 需要实现栈或队列功能

##### 五、线程安全
- 两者均非线程安全，多线程环境下需通过以下方式实现同步
```java
//方式1 简单同步，性能较低 
Collections.synchronizedList(); 

//方式2 写时复制，适合读多写少场景
CopyOnWriteArrayList
```
##### 六、代码示例
```java

//一、声明和赋值
//默认构造（初始容量10）
ArrayList<Integer> arrayList = new ArrayList<Integer>();   
//指定初始容量，防止频繁扩容 
ArrayList<Integer> arrayList1 = new ArrayList<>(20);  
//从数组初始化  
List<Integer> fromArrayList = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10); 
//或直接传入List数组
ArrayList<Integer> fromOtherList = new ArrayList<>(fromArrayList); 


//二、增删改查
//1.add增加元素
//追加到末尾,返回boolean表示是否添加成功
boolean add = list.add(1);
//追加到索引, index越界会报IndexOutOfBoundsException
list.add(0, 2);
//1）添加装箱类
Integer i = new Integer("1"); //不推荐 java9移除
Integer i = Integer.valueOf(1); //推荐
list.add(i);
//2）批量添加
//追加到末尾
list.addAll(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9));
//追加到索引
list.addAll(2, Arrays.asList(1, 2, 3, 4, 5));

//2.remove删除元素
//按照索引删除
list.remove(1);
//按照值删除，仅删除首次出现的元素，返回值为是否删除成功
boolean remove = list.remove(Integer.valueOf(10087));
//清空list
list.clear();

//3.modify修改元素
//修改指定索引的内容
list.set(0, Integer.valueOf(10087));

//4.查询query
//获取元素按照索引，索引越界会报异常
Integer a = list.get(1);
//检查是否存在
boolean contains = list.contains(2);
//查找索引，首次出现的位置
int firstIndex = list.indexOf(2);
//查找索引，最后一次出现位置
int lastIndex = list.lastIndexOf(2);


//三、遍历Traversal
//普通for循环遍历
for(int i = 0;i < list.size;i++){
   System.out.println(list.get(i));
}
//增强for循环
for(Integer i : list){
   System.out.println("%d",i);
}
//迭代器遍历
Iterator<Integer> iterator = list.iterator();
while(iterator.hasNext()){
   System.out.println(iterator.next());
}


//四、其他常用工具
//获取list大小
int size = list.size();
//检查list是否为空
boolean empty = list.isEmpty();
//list转数组
Object[] objects = list.toArray();
//list转数组但指定类型
Integer[] nums = list.toArray(new Integer[list.size]);
//预扩容优化(减少动态扩展次数)
list.ensureCapacity(100);

```

```java
//LinkedList使用
List<String> linkedList = new LinkedList<>();

//添加元素
linkedList.add("Apple");
linkedList.add("Banana");

//头部插入
linkedList.addFirst("Cherry");
System.out.println("头部插入后："+ linkedList);

//尾部插入
linkedList.addLast("Date");
System.out.println("尾部插入后："+ linkedList);

//删除头部元素
linkedList.removeFirst();
System.out.println("删除头部后："+ linkedList);

//删除尾部元素
linkedList.removeLast();
System.out.println("删除尾部后："+ linkedList);

//使用LinkedList 实现栈
LinkedList<String> stack = new LinkedList<>();
stack.push("A"); //入栈
stack.push("B");
System.out.println(stack.pop()); //出栈

//使用LinkedList 实现队列
LinkedList<String> queue = new LinkedList<>();
queue.offer("A"); //入队
queue.offer("B");
System.out.println(queue.poll()); //出队
```