---
创建时间: 2025-04-09 12:25:33
作者: wangxiaoming
tags:
  - Stack
  - Java
---

Stack数据结构特性可查看 [[常见数据结构#3)栈（Stack）]]

#### 一、java中栈的使用
##### 1)ArrayDeque使用
###### 优势：
- 高效性：ArrayDeque 基于动态数组，内存连续，访问速度快。
- 灵活性：同一Deque 实例可同时支持队列和栈操作。
- 兼容性：适用于单线程场景，若需线程安全可改用 ConcurrentLinkedDeque。

###### 使用
```java
Deque<Integer> arrayDequeStack = new ArrayDeque<>(); //推荐使用ArrayDeque(数组实现，性能高)  
Deque<Integer> linkedDequeStack = new LinkedList<>(); //链表实现，适合频繁插入/删除  
  
//1.操作压栈  
arrayDequeStack.push(1);  
arrayDequeStack.push(2);  
arrayDequeStack.push(3);  
System.out.println("arrayDeque 当前栈内情况 "+ arrayDequeStack);  
System.out.println("arrayDeque 压栈后出栈1次结果 " + arrayDequeStack.peek());  
System.out.println("arrayDeque 压栈后出栈2次结果 " + arrayDequeStack.peek());  
System.out.println("-----------------------------------------------------");  
  
//2.操作弹栈  
System.out.println("arrayDeque 压栈后弹栈1次结果 " + arrayDequeStack.pop());  
System.out.println("arrayDeque 当前栈内情况 "+ arrayDequeStack);  
System.out.println("arrayDeque 压栈后出栈2次结果 " + arrayDequeStack.pop());  
System.out.println("arrayDeque 当前栈内情况 "+ arrayDequeStack);  
System.out.println("arrayDeque 压栈后出栈3次结果 " + arrayDequeStack.pop());  
System.out.println("arrayDeque 当前栈内情况 "+ arrayDequeStack);  
System.out.println("-----------------------------------------------------");  
  
//3.操作查看栈顶  
arrayDequeStack.peek();  
System.out.println("arrayDeque 查看栈顶 " + arrayDequeStack.peek());  
arrayDequeStack.push(4);  
System.out.println("arrayDeque 压栈1次后 查看栈顶 " + arrayDequeStack.peek());  
arrayDequeStack.push(5);  
System.out.println("arrayDeque 压栈2次后 查看栈顶 " + arrayDequeStack.peek());  
System.out.println("-----------------------------------------------------");  
  
//4.判空  
boolean empty = arrayDequeStack.isEmpty();  
System.out.println("arrayDeque 判空 " + empty);  
arrayDequeStack.clear();  
System.out.println("arrayDeque 清栈后 判空 " + true);  
System.out.println("-----------------------------------------------------");
```

##### 2）Stack 使用（继承自Vector）
###### 缺点
- 同步开销：所有方法均为同步（线程安全），单线程下性能较差。
- 设计过时：API不够乐观，如 search 方法返回基于栈底的索引。

###### 使用
```java
Stack<Integer> stack = new Stack<>(); //声明

//操作和其他基本一样
stack.pop();
stack.peek();
stack.push(1);
stack.isEmpty();
stack.clear();
```

##### 3）自定义栈实现
###### 使用数组实现
```java
public class ArrayStack<E> {
   private E[] elements; //定义数组
   private top = -1; //定义栈指针
   public ArrayStack(int capacity){
       elements = (E[]) new Object[capacity];
   }
   //压栈
   private void push(E item){
       if(top == elements.length - 1) resize();
       elements[++top] = item;
   } 
   //出栈
   private E peek(){
       return elements[top];
   }
   //弹栈
   private E pop(){
       if(top == -1) throw new EmptyStackException();
       E item = elements[pop]; //返回栈顶元素
       elements[pop--] = null; //清空栈顶并递减指针
       return item;
   }
   private resize(){
      //扩容逻辑
   }
}
```

###### 使用链表实现
```java
public class LinkedStack<E> {

   private Node<E> top; //栈顶节点
   //链表节点
   public static class Node<E> {
      private E data;
      private Node<E> next;
      public Node<E>(E data){
         this.data = data;
      }
   }
   
   //压栈
   private void push(E item){
      Node<E> newNode = new Node<>(item); //创建新节点
      newNode.next = top; //指定新节点next为现在的top
      top = newNode; //指定新的top为当前节点（直接引用指定）
   }
   
   //出栈
   private E peek(){
      return top.data;
   }
   
   //弹栈
   private E pop(){
     if(pop == null) throw new EmptyStackException();
     E item = pop.data;
     top = top.next; //新top和top的next
     return item;
   }
}
```