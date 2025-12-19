---
创建时间: "2025-12-19 16:32:06"
作者: wangxiaoming
tags:
---
![[Java Error.png]]

#### 一、加载阶段
这个阶段的主要任务是通过类名找到对应的 `.class`文件并读入内存。
##### 1）`ClassNotFoundException`
 1️⃣ 说明
最常见的加载异常。JVM在Classpath中找不到指定的类文件、
 2️⃣ 触发场景
 ```java
 Class.forName("com.example.NonExistentClass")
 ```
##### 2) `SecurityException`
1️⃣ 说明
安全管理器（`SecurityManager`）阻止了类加载操作。
2️⃣ 触发场景
在Applet或沙箱环境中，安全管理器禁止读取特定路径的文件。

#### 二、链接阶段
链接阶段分为验证、准备、解析三步。
##### a) 验证
确保字节码是合法、安全的，符合JVM规范。
######  1）`VerifyError`
1️⃣ 说明
字节码验证失败。编译器生成的代码不符合JVM规范（如栈帧状态不一致）。
2️⃣ 触发场景
通常由编译器bug或不兼容的字节码工具引起。

###### 2）`SecurityException`
1️⃣ 说明
验证过程中发现代码违反安全约束。

##### b)准备
为类的静态变量分配内存并设置**默认初始值**（零值，如 `int`为0，`Object`为 `null`）。
###### 1)`OutOfMemoryError`
1️⃣ 说明
如果在准备阶段需要为大量静态变量分配内存，而堆内存不足。
2️⃣ 触发场景
极其罕见，通常JVM内存不足更早暴露。

##### c) 解析
将常量池中的**符号引用**替换为**直接引用**（内存地址指针）
###### 1)`NoSuchMethodError`
1️⃣ 说明
当代码中引用的一个方法在目标类中不存在时。
2️⃣ 触发场景
编译时存在该方法，但运行时使用的类版本中该方法被删除或改名。

###### 2)`NoSuchFieldError`
1️⃣ 说明
类似 `NoSuchMethodError`，但针对字段。
2️⃣ 触发场景
编译时存在该字段，但运行时使用的类版本中该字段被删除或改名。

###### 3）`IllegalAccessError`
1️⃣ 说明
访问权限不足。例如，试图访问一个 `private`方法
2️⃣ 触发场景
编译时可能通过反射绕过检查，但运行时JVM严格检查。

#### 三、初始化阶段
执行类的静态变量赋值和静态代码块（即 `<clinit>`方法）。

##### 1)`ExceptionInInitializerError`
1️⃣ 说明
**初始化失败**。静态代码块或静态变量赋值中抛出了任何异常（`Exception`）。
2️⃣ 触发场景
```java
static { int i = 1 / 0; }
```
###### 2）`OutOfMemoryError`
1️⃣ 说明
执行静态代码块时（例如在静态块中创建了巨大的数组）导致内存耗尽。
2️⃣ 触发场景
```java
static int[] hugeArray = new int[Integer.MAX_VALUE];
```

#### 四、使用阶段
类初始化成功后，就可以正常使用了。这个阶段会产生我们最常用的异常。
##### 1）`NoClassDefFoundError`
1️⃣ 说明
**类定义丢失**。由 `ExceptionInInitializerError`引发，导致类无法再被使用。
2️⃣ 触发场景
同上

##### 2）`NoSuchMethodError`
1️⃣ 说明
与方法调用相关，发生在运行时。比链接阶段的解析错误更常见，因为解析可能发生在首次调用时。
2️⃣ 触发场景
同上

##### 3）`AbstractMethodError`
1️⃣ 说明
调用了一个抽象方法或接口方法，但具体的实现类没有提供实现。
2️⃣ 触发场景
常见于版本不匹配，如接口新增了抽象方法，但实现类未更新。

##### 4) `UnsatisfiedLinkError`
1️⃣ 说明
找不到声明的本地方法（`native`method）对应的本地库（.dll/.so）。
2️⃣ 触发场景
```java
System.loadLibrary("my_native_lib") //失败，或库中缺少对应方法。
```

##### 5) `ClassCastException`
1️⃣ 说明
强制类型转换失败。
2️⃣ 触发场景
```java
Object obj = "string"; Integer i = (Integer) obj;
```

##### 6) `IllegalAccessError`
1️⃣ 说明 
同链接阶段，但在使用阶段也可能发生（如通过接口引用调用包级私有的实现类方法）。

##### `7) ArrayStoreException`
1️⃣ 说明
试图将错误类型的对象存储到对象数组中。
2️⃣ 触发场景
```java
Object[] arr = new String[10]; arr[0] = new Integer(5)
```

##### `8) NullPointerException`
1️⃣ 说明
最常见异常。试图在 `null`引用上调用方法或访问字段。
2️⃣ 触发场景
```java
String s = null;
s.length();
```

#### 五、卸载阶段
JVM 垃圾回收器回收不再被使用的类。这个阶段通常不会产生直接的异常。如果一个类无法被卸载，可能是因为：
- 类的 `ClassLoader`仍然存活。
- 该类创建的对象仍在被引用。

#### 六、跨阶段的通用错误
这些错误可能在任何阶段发生，因为它们是JVM运行时资源耗尽的表现。

##### 1)`OutOfMemoryError`
1️⃣ 说明
**Java堆溢出**：创建对象时堆空间不足。  
**元空间溢出**：加载的类过多，元空间（Metaspace）不足。  
**栈溢出**：`StackOverflowError`，通常是递归调用过深。

##### `2)StackOverflowError`
1️⃣ 说明
线程请求的栈深度大于虚拟机所允许的深度。


