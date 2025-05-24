---
创建时间: 2025-05-19 17:27:34
作者: wangxiaoming
tags:
  - 链表
  - 算法
---
##### 1）题目

给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 _如果链表无环，则返回 `null`。_

如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。

**不允许修改** 链表。

**示例 1：**

![](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png)

**输入：**head = [3,2,0,-4], pos = 1
**输出：**返回索引为 1 的链表节点
**解释：**链表中有一个环，其尾部连接到第二个节点。

**示例 2：**

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png)

**输入：**head = [1,2], pos = 0
**输出：**返回索引为 0 的链表节点
**解释：**链表中有一个环，其尾部连接到第一个节点。

**示例 3：**

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png)

**输入：**head = [1], pos = -1
**输出：**返回 null
**解释：**链表中没有环。

**提示：**

- 链表中节点的数目范围在范围 `[0, 104]` 内
- `-105 <= Node.val <= 105`
- `pos` 的值为 `-1` 或者链表中的一个有效索引
##### 3)代码示例
```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode pos = head;
        Set<ListNode> mySet = new HashSet<ListNode>();
        while(pos != null){
            if(!mySet.contains(pos)){
                mySet.add(pos);
            } else {
                return pos;
            }
            pos = pos.next;
        }
        return null;
    }

}
```