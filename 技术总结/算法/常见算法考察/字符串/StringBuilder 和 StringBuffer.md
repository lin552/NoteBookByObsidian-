---
创建时间: 2025-04-12 17:56:36
作者: wangxiaoming
tags:
  - StringBuffer
  - StringBuilder
---

#### 一、核心区别与特性对比

| **特性**​     | ​**StringBuffer**​              | ​**StringBuilder**​    |
| ----------- | ------------------------------- | ---------------------- |
| ​**线程安全性**​ | 线程安全（方法用 `synchronized` 修饰）<br> | 非线程安全（无同步锁）<br>        |
| ​**性能**​    | 较低（同步锁开销）                       | 更高（无锁，单线程效率提升约50%）<br> |
| ​**适用场景**​  | 多线程环境（如日志记录、并发拼接）<br>           | 单线程高频操作（如循环拼接、动态SQL生成） |
| ​**可变性**​   | 可变（内部字符数组可动态扩容）<br>             | 可变（同StringBuffer）      |
#### 二、面试题考察
##### 1）`StringBuffer` 如何保证线程安全？
所有修改字符串的方法（如 `append()`、`insert()`、`delete()`）均通过`synchronized` 关键字实现同步锁，确保同一时间仅一个线程可操作对象
```java
public synchronized StringBuffer append(String str){
    super.append(str);
    return this;
}
```
代价：频繁的锁竞争会导致性能损耗，尤其在单线程场景下
##### 2）为什么String不可变，而`StringBuffer/StringBuilder`可变
`String`的字符数组被`final`修饰，每次修改生成新对象；后两者通过动态调整字符数组实现原地修改
##### 3）单线程下能否用`StringBuffer`
可以，但性能低于`StringBuilder`
##### 4）拼接字符串时如何选择？
少量拼接用`+`（编译器优化为`StringBuilder`），大量操作显式使用`StringBuiler`
#### 三、使用建议与优化
##### 1）场景选择
优先选`StringBuilder`：单线程操作（如循环拼接、`JSON/XML`构建）
优先选 `StringBuffer`：多线程共享变量且涉及修改（如全局日志缓存区）
##### 2）性能优化
**预设容量**：避免频繁扩容（默认初始容量16），例如`new StringBuilder(100)`
**避免混用** **`+` 运算符**：`+` 会隐式生成多个临时对象，优先使用`append()`
##### 3）线程安全替代方案
若需在多线程中使用`StringBuilder`，需手动加锁（如`synchronized`代码块）

#### 四、常用方法示例
```java
System.out.println("------------StringBuilder使用---------------------------------");  
StringBuffer stringBuffer = new StringBuffer("Hello");  
System.out.println("StringBuilder 使用 string "+stringBuffer);  
stringBuffer.append("World").append("!"); //追加  
System.out.println("StringBuffer 使用 append String "+stringBuffer);  
stringBuffer.insert(5," AAA "); //插入  
System.out.println("StringBuffer 使用 insert String "+stringBuffer);  
stringBuffer.delete(5,10); //删除  
System.out.println("StringBuffer 使用 delete String "+stringBuffer);  
stringBuffer.replace(0,5,"olleH"); //替换  
System.out.println("StringBuffer 使用 replace String "+stringBuffer);  
stringBuffer.reverse(); //反转  
System.out.println("StringBuffer 使用 reverse String "+stringBuffer);  
  
  
System.out.println("------------StringBuilder使用---------------------------------");  
StringBuilder stringBuilder = new StringBuilder("Hello");  
for (int i = 0; i < 1000; i++) {  
    stringBuilder.append(i);  
}  
System.out.println("StringBuilder 使用 单线程效率 "+stringBuilder);
```