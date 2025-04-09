---
创建时间: 2025-04-09 19:13:18
作者: wangxiaoming
tags:
  - Java
---
#### 一、== 操作符的行为
##### 1)基础数据类型的比较
== 直接比较变量的值是否相等
```java
int a = 10,b = 10;
System.out.println(a == b); //返回true
```
##### 2)引用数据类型比较
== 比较的是对象在堆内存中的引用地址（即是否为同一个对象），而非对象内容
```java
String s1 = new String("hello");
String s2 = "hello";
```
##### 3)特殊场景
###### 包装类缓存机制 
如Integer 在 -128~127 范围内会复用对象，此时 == 可能返回 true
```java
Integer a = 100,b = 100;
System.out.println(a == b);// true
```
##### 字符串常量池
字面量赋值的字符串可能复用对象，但new String() 会创建新对象。
[[Java 字符串 String#3)字符串常量池复用机制]]

#### 二、equals 方法的行为
##### 1）默认实现
未重写时，equals 与 == 行为一致，比较引用地址
```java
Object obj1 = new Object();
Object obj2 = obj1;
System.out.println(obj1.equals(obj2)); // true 同一对象
```

##### 2）重写后的实现
大多数常用类（如 String、Integer）重写了 equals()，使其比较对象内容而非地址
```java
String s1 = new String("hello");
String s2 = "hello";
System.out.println(s1.equals(s2)); //true 

Integer a = Integer.valueOf(1);
Integer b = 1;
System.out.println(a.equals(b)); //true 
```