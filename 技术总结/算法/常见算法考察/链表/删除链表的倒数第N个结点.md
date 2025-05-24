---
创建时间: 2025-05-20 08:15:34
作者: wangxiaoming
tags:
  - 链表
  - 算法
---
##### 1）题目

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

**示例 1：**

![](https://assets.leetcode.com/uploads/2020/10/03/remove_ex1.jpg)

**输入：**head = [1,2,3,4,5], n = 2
**输出：**[1,2,3,5]

**示例 2：**

**输入：**head = [1], n = 1
**输出：**[]

**示例 3：**

**输入：**head = [1,2], n = 1
**输出：**[1]

**提示：**

- 链表中结点的数目为 `sz`
- `1 <= sz <= 30`
- `0 <= Node.val <= 100`
- `1 <= n <= sz`

##### 2）解题思路
先遍历链表获得链表长度，再for循环找到(length - n + 1 位置)，将该指针的next改为next.next 就是删除了节点
##### 3）代码示例
```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        //创建一个ListNode方便返回的时候直接拿来用
        ListNode dummy = new ListNode(0,head);
        //获取链表长度
        int length = getlength(head);
        ListNode cur = dummy;
        //遍历找到倒数第n个节点
        for(int i = 1; i < length - n + 1;++i){
            cur = cur.next;
        }
        cur.next = cur.next.next; //删除节点
        ListNode ans = dummy.next; //dummy.next 就是head 开头的
        return ans;
    }
    //遍历获取链表长度
    public int getlength(ListNode head){
        int length = 0;
        while(head != null){
            ++length;
            head = head.next;
        }
        return length;
    }
}

// 快慢指针法
class Solution {

    public ListNode removeNthFromEnd(ListNode head, int n) {

        ListNode dummy = new ListNode(0,head);
        ListNode fast = dummy;
        ListNode slow = dummy;

        while(n > 0){
           fast = fast.next;
           n--;
        }

        while(fast.next != null){
            slow = slow.next;
            fast = fast.next;
        }

        slow.next = slow.next.next;
        return dummy.next;
    }

}
```
