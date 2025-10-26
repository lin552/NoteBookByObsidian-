---
创建时间: "2025-06-24 16:37:58"
作者: wangxiaoming
tags:
---

#### **一、`BFS`（广度优先搜索）​**​

`BFS` 使用**队列（Queue）​**实现，按层遍历节点，适合解决层序相关问题（如计算树的深度、最短路径、分层打印等）。
##### ​**1. 基础 `BFS`（不分层）​**​
遍历顺序：根节点 → 同一层所有节点 → 下一层所有节点 → ... 直到叶子节点。
```java
public class BFSTraversal {
    public static void bfs(TreeNode root) {
        if (root == null) return;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root); // 根节点入队

        while (!queue.isEmpty()) {
            TreeNode node = queue.poll(); // 取出队首节点
            System.out.print(node.val + " "); // 访问当前节点

            // 左子节点入队（非空时）
            if (node.left != null) {
                queue.offer(node.left);
            }
            // 右子节点入队（非空时）
            if (node.right != null) {
                queue.offer(node.right);
            }
        }
    }

    public static void main(String[] args) {
        // 构建测试二叉树：
        //       1
        //      / \
        //     2   3
        //    / \
        //   4   5
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.right = new TreeNode(3);
        root.left.left = new TreeNode(4);
        root.left.right = new TreeNode(5);

        System.out.println("BFS 遍历结果：");
        bfs(root); // 输出：1 2 3 4 5
    }
}
```
##### **2. 分层 `BFS`（记录每层节点）​**​
若需要区分每一层的节点（如计算树的层数或每层平均值），可通过记录**当前层的节点数量**来实现分层。
```java
public static void bfsWithLevel(TreeNode root) {
    if (root == null) return;

    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    int level = 0;

    while (!queue.isEmpty()) {
        int levelSize = queue.size(); // 当前层的节点数
        System.out.print("第 " + (++level) + " 层: ");

        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            System.out.print(node.val + " ");

            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        System.out.println(); // 换行分隔层
    }
}

// 测试输出：
// 第 1 层: 1 
// 第 2 层: 2 3 
// 第 3 层: 4 5 
```
#### ​**三、`DFS`（深度优先搜索）​**​
`DFS` 使用**递归**或**栈（Stack）​**实现，沿树的深度遍历，分为三种顺序：
- ​**前序遍历**​：根 → 左 → 右
- ​**中序遍历**​：左 → 根 → 右（二叉搜索树的中序遍历结果为有序序列）
- ​**后序遍历**​：左 → 右 → 根
##### **1. 递归实现 `DFS`**​
递归是最直观的方式，但需注意递归深度限制（对于极深的树可能栈溢出）。
###### ​**前序遍历（根→左→右）**
```java
public static void dfsPreOrderRecursive(TreeNode root) {
    if (root == null) return;
    System.out.print(root.val + " "); // 访问根节点
    dfsPreOrderRecursive(root.left);  // 递归左子树
    dfsPreOrderRecursive(root.right); // 递归右子树
}
```
###### **中序遍历（左→根→右）**
```java
public static void dfsInOrderRecursive(TreeNode root) {
    if (root == null) return;
    dfsInOrderRecursive(root.left);  // 递归左子树
    System.out.print(root.val + " "); // 访问根节点
    dfsInOrderRecursive(root.right); // 递归右子树
}
```
###### **后序遍历（左→右→根）​**
```java
public static void dfsPostOrderRecursive(TreeNode root) {
    if (root == null) return;
    dfsPostOrderRecursive(root.left);  // 递归左子树
    dfsPostOrderRecursive(root.right); // 递归右子树
    System.out.print(root.val + " "); // 访问根节点
}
```

##### ​**2. 迭代实现 `DFS`（使用栈）​**​
递归的本质是系统栈，迭代实现需手动维护栈结构，关键是模拟递归的访问顺序。
###### **前序遍历（迭代）​**
```java
import java.util.Deque;
import java.util.Stack;

public static void dfsPreOrderIterative(TreeNode root) {
    if (root == null) return;
    Deque<TreeNode> stack = new LinkedList<>();
    stack.push(root); // 根节点入栈

    while (!stack.isEmpty()) {
        TreeNode node = stack.pop(); // 弹出栈顶节点
        System.out.print(node.val + " "); // 访问根节点

        // 右子节点先入栈（后处理），左子节点后入栈（先处理）
        if (node.right != null) stack.push(node.right);
        if (node.left != null) stack.push(node.left);
    }
}
```
###### **中序遍历（迭代）​**
```java
public static void dfsInOrderIterative(TreeNode root) {
    Deque<TreeNode> stack = new LinkedList<>();
    TreeNode curr = root;

    while (curr != null || !stack.isEmpty()) {
        // 一直向左走，直到叶子节点
        while (curr != null) {
            stack.push(curr);
            curr = curr.left;
        }
        // 此时左子树已处理完，弹出栈顶（最左节点）
        curr = stack.pop();
        System.out.print(curr.val + " "); // 访问根节点
        // 转向右子树
        curr = curr.right;
    }
}
```
###### **后序遍历（迭代）​**​
后序遍历的迭代实现较复杂，需区分“访问根节点”的时机（左右子树均处理完后）。常见方法有两种：
- ​**标记法**​：用一个变量记录是否已访问过当前节点的子节点。
- ​**双栈法**​：用两个栈分别处理左右子树的顺序。
​**标记法实现：​**
```java
public static void dfsPostOrderIterative(TreeNode root) {
    if (root == null) return;
    Deque<TreeNode> stack = new LinkedList<>();
    Stack<Integer> output = new LinkedList<>(); // 记录访问顺序
    stack.push(root);

    while (!stack.isEmpty()) {
        TreeNode node = stack.pop();
        output.push(node.val); // 先压入输出栈（最终逆序输出）

        // 左子节点先压入（保证后处理），右子节点后压入（先处理）
        if (node.left != null) stack.push(node.left);
        if (node.right != null) stack.push(node.right);
    }

    // 输出栈逆序即为后序（根→右→左 → 逆序后为左→右→根）
    while (!output.isEmpty()) {
        System.out.print(output.pop() + " ");
    }
}
```
​**双栈法实现（更直观）：​**
```java
public static void dfsPostOrderIterativeTwoStacks(TreeNode root) {
    if (root == null) return;
    Deque<TreeNode> stack1 = new LinkedList<>();
    Deque<TreeNode> stack2 = new LinkedList<>();
    stack1.push(root);

    while (!stack1.isEmpty()) {
        TreeNode node = stack1.pop();
        stack2.push(node); // 先压入 stack2（顺序：根→右→左）

        if (node.left != null) stack1.push(node.left);
        if (node.right != null) stack1.push(node.right);
    }

    // stack2 弹出顺序为左→右→根
    while (!stack2.isEmpty()) {
        System.out.print(stack2.pop().val + " ");
    }
}
```

#### **四、`BFS` 与 `DFS` 对比**​

|​**维度**​|​**BFS（广度优先）​**​|​**DFS（深度优先）​**​|
|---|---|---|
|​**核心数据结构**​|队列（Queue）|栈（Stack）或递归（系统栈）|
|​**遍历顺序**​|按层横向扩展（根→同一层→下一层）|沿深度纵向深入（根→左/右→更深层）|
|​**空间复杂度**​|O(n)（最坏情况队列存一层节点）|O(h)（h为树高，递归栈深度）|
|​**适用场景**​|层序问题（最短路径、分层统计）|路径搜索（如二叉树最大路径和）、前/中/后序问题|
|​**时间复杂度**​|O(n)（每个节点访问一次）|O(n)（每个节点访问一次）|

#### ​**五、总结**​
- ​`BFS` 适合需要按层处理的场景（如计算树的层数、寻找最短路径），用队列实现。
- ​`DFS`​ 适合需要深度优先探索的场景（如路径搜索、前/中/后序遍历），可用递归或栈实现。
- 实际开发中，`DFS` 的递归实现更简洁，但需注意栈溢出问题（极深树时改用迭代）；`BFS` 的队列实现更直观，适合分层问题。