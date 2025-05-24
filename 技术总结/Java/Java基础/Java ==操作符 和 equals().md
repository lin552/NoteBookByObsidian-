---
创建时间: 2025-04-09 19:13:18
作者: wangxiaoming
tags:
  - Java
---
#### 一、`==` 操作符的行为
##### 1)基础数据类型的比较
`==` 直接比较变量的值是否相等
```java
int a = 10,b = 10;
System.out.println(a == b); //返回true
```
##### 2)引用数据类型比较
`==` 比较的是对象在堆内存中的引用地址（即是否为同一个对象），而非对象内容
```java
String s1 = new String("hello");
String s2 = "hello";
```
##### 3)特殊场景
###### 包装类缓存机制 
如`Integer` 在 -128~127 范围内会复用对象，此时 `==` 可能返回 `true`
```java
Integer a = 100,b = 100;
System.out.println(a == b);// true
```
##### 字符串常量池
字面量赋值的字符串可能复用对象，但`new String()` 会创建新对象。
[[Java 字符串 String#3)字符串常量池复用机制]]

#### 二、`equals()`  方法的行为
##### 1）默认实现
未重写时，`equals` 与 `==` 行为一致，比较引用地址
```java
Object obj1 = new Object();
Object obj2 = obj1;
System.out.println(obj1.equals(obj2)); // true 同一对象
```

##### 2）重写后的实现
大多数常用类（如 `String`、`Integer`）重写了 `equals()`，使其比较对象内容而非地址
```java
String s1 = new String("hello");
String s2 = "hello";
System.out.println(s1.equals(s2)); //true 

Integer a = Integer.valueOf(1);
Integer b = 1;
System.out.println(a.equals(b)); //true 
```

##### 3）自定义类
需手动重写`equals()` 以实现内容比较，否则默认行为与 `==` 相同
```java
class Person {
   String name;
   //重写equals比较name属性
   @Override
   public boolean equals(Object obj){
      return obj instanceof Person && ((Person)obj).name.equals(this.name);
   }
}
```

#### 三、`==` 和 `equals()` 核心区别
| ​**维度**​   | ​**`==`**​       | ​**`equals()`**​ |
| ---------- | ---------------- | ---------------- |
| ​**比较对象**​ | 基本类型值或引用地址       | 对象内容（需重写方法）      |
| ​**适用场景**​ | 检查是否为同一对象实例      | 检查逻辑上的内容相等性      |
| ​**默认行为**​ | 引用类型比较地址，基本类型比较值 | 未重写时与 `==` 相同    |
| ​**性能**​   | 高效（直接地址或值比较）     | 可能较慢（需遍历内容或计算哈希） |
#### 四、使用建议与注意事项
###### 1）何时使用 `==` 
- 验证对象是否为同一个实例(如单例模式)
- 基本类型值的快速比较

##### 2）何时使用 `equals()`
- 需要逻辑内容相等的场景(如字符串、集合元素比较)
- 自定义类需重写 `equals()` 并同步重写 `hashCode()` (如用于HashMap键值)
##### 3）特殊值的处理
- `null` 安全：调用 `equals()` 前需判空，避免 `NullPointerException` 
- 浮点数比较：避免直接 `==` ，推荐使用误差范围（如 `Math.abs(a-b) < 1e-6`）