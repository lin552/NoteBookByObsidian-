---
创建时间: 2025-04-10 12:48:27
作者: wangxiaoming
tags:
  - 算法
  - 链表
---
#### 一、题目
给你链表的头节点 `head` ，每 `k` 个节点一组进行翻转，请你返回修改后的链表。

`k` 是一个正整数，它的值小于或等于链表的长度。如果节点总数不是 `k` 的整数倍，那么请将最后剩余的节点保持原有顺序。

你不能只是单纯的改变节点内部的值，而是需要实际进行节点交换。

**示例 1：**

![](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex1.jpg)

**输入：**head = [1,2,3,4,5], k = 2
**输出：**[2,1,4,3,5]

**示例 2：**

![](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex2.jpg)

**输入：**head = [1,2,3,4,5], k = 3
**输出：**[3,2,1,4,5]

**提示：**

- 链表中的节点数目为 `n`
- `1 <= k <= n <= 5000`
- `0 <= Node.val <= 1000`

#### 二、代码实践
```java
public ListNode reverseKGroup(ListNode head,int k){
   //检测当前数组是否足够k个节点
   ListNode curr = head;
   int count = 0;
   while(curr != null && count < k){
      curr = curr.next;
      count++;
   }

   //满足k个则反转当前组，否则直接返回
   if(count == k){
      ListNode reversedHead = reverseSubList(head,curr); //反转当前组
      //递归处理剩余链表，并将当前组的尾部与之连接
      head.next = reverseKGroup(curr,k);
      return reversedHead
   }
   return head;
}

// 反转列表方法
private ListNode reverseSubList(ListNode start,ListNode end){
   ListNode prev = null,curr = start;
   while(curr != end){
      ListNode next = curr.next;
      prev = curr;
      curr = next;
   }
   return prev;
}
```