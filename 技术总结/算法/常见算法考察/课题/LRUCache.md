---
创建时间: 2025-06-09 18:26:05
作者: wangxiaoming
tags:
  - LruCache
---
#### 一、题目
请你设计并实现一个满足`[LRU (最近最少使用) 缓存]`(https://baike.baidu.com/item/LRU) 约束的数据结构。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以 **正整数** 作为容量 `capacity` 初始化 `LRU` 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字 `key` 已经存在，则变更其数据值 `value` ；如果不存在，则向缓存中插入该组 `key-value` 。如果插入操作导致关键字数量超过 `capacity` ，则应该 **逐出** 最久未使用的关键字。

函数 `get` 和 `put` 必须以 `O(1)` 的平均时间复杂度运行。

**示例：**

**输入**
["LRUCache", "put", "put", "get", "put", "get", "put", "get", "get", "get"]
[[2], [1, 1], [2, 2], [1], [3, 3], [2], [4, 4], [1], [3], [4]]
**输出**
[null, null, null, 1, null, -1, null, -1, 3, 4]

**解释**
```java
LRUCache lRUCache = new LRUCache(2);
lRUCache.put(1, 1); // 缓存是 {1=1}
lRUCache.put(2, 2); // 缓存是 {1=1, 2=2}
lRUCache.get(1);    // 返回 1
lRUCache.put(3, 3); // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
lRUCache.get(2);    // 返回 -1 (未找到)
lRUCache.put(4, 4); // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
lRUCache.get(1);    // 返回 -1 (未找到)
lRUCache.get(3);    // 返回 3
lRUCache.get(4);    // 返回 4
```


**提示：**

- `1 <= capacity <= 3000`
- `0 <= key <= 10000`
- `0 <= value <= 105`
- 最多调用 `2 * 105` 次 `get` 和 `put`

#### 二、解题思路

哈希表 + 双向链表 ，通过哈希表去重，通过双向链表维护最近的使用，将最近使用的移动到双向链表头部

#### 三、代码
```java
class LRUCache {

// 1.设置双向链表结构
    class DLinkedNode {
        int key;
        int value;
        DLinkedNode prev;
        DLinkedNode next;
        public DLinkedNode() {

        }
        public DLinkedNode(int _key,int _value){
            key = _key;
            value = _value;
        }
    }

//2.创建哈希表
    private Map<Integer,DLinkedNode> cache = new LinkedHashMap<Integer,DLinkedNode>();
    private int size;
    private int capacity;
    // 3.设置虚拟首尾指针
    private DLinkedNode head,tail;

    public LRUCache(int capacity) {
        this.size = 0;
        this.capacity = capacity;
        // 初始化首尾指针
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        DLinkedNode node = cache.get(key);
        //如果key不存在 返回-1
        if(node == null){
            return -1;
        }
        //如果key存在移动到首部
        moveToHead(node);
        return node.value;
    }

    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);
        //如果key不存在就创建一个
        if(node == null){
            DLinkedNode newNode = new DLinkedNode(key,value);
            //添加到哈希表
            cache.put(key,newNode);
            //添加到头部
            addToHead(newNode);
            ++size;
            //如果超出容量
            if(size > capacity){
                //删除末尾节点
               DLinkedNode tail = removeTail();
               //哈希表中移除
               cache.remove(tail.key);
               --size;
            }
        } else {
            //如果key存在,先通过哈希表定位，再修改value,并移动头部
            node.value = value;
            moveToHead(node);
        }
    }

    private void addToHead(DLinkedNode node){
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }

    private void removeNode(DLinkedNode node){
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void moveToHead(DLinkedNode node){
        //先移除
        removeNode(node);
        //再加入到头部
        addToHead(node);
    }

    private DLinkedNode removeTail(){
        DLinkedNode res = tail.prev;
        removeNode(res);
        return res;
    }
}
```