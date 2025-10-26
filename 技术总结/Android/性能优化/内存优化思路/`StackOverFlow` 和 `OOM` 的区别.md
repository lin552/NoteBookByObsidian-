---
创建时间: 2025-07-01 22:30:25
作者: wangxiaoming
tags:
  - stackOverflow
  - OOM
---

在Java/Kotlin虚拟机（`JVM`）中，`​**StackOverflowError`（栈溢出）​和`OutOfMemoryError`（`OOM`，内存溢出）​**是两种典型的运行时错误，但它们的触发原因、发生位置和解决方法有本质区别。以下是两者的详细对比：

#### 一、核心区别总结
| **维度**​      | ​**StackOverflowError（栈溢出）​**​                                                                      | ​**OutOfMemoryError（OOM，内存溢出）​**​                                                                             |
| ------------ | --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| ​**触发位置**​   | 虚拟机的**栈内存（Stack）​**​：存放方法调用的栈帧（局部变量、操作数栈、动态链接等）。                                                    | 虚拟机的**堆内存（Heap）​**​：存放对象实例（如`Bitmap`、`String`、自定义对象等）。                                                        |
| ​**核心原因**​   | 栈空间被耗尽：方法调用层次过深（如无限递归、深层嵌套调用），导致栈帧数量超过栈的最大容量。                                                       | 堆空间被耗尽：对象实例过多（如大量未释放的对象、大对象直接分配）或堆内存分配失败（如尝试创建超大对象）。                                                          |
| ​**典型场景**​   | - 无限递归（如`void foo() { foo(); }`）；  <br>- 深层方法调用链（如多层嵌套的工具方法）；  <br>- JVM栈内存过小（默认栈大小通常为1MB~几MB）。     | - 加载超大对象（如未压缩的高清图片）；  <br>- 内存泄漏（对象无法回收，累积占满堆）；  <br>- 堆内存过小（如`-Xmx`参数设置过小）；  <br>- 大量临时对象（如循环中创建大量`String`）。 |
| ​**错误日志特征**​ | 日志明确提示`StackOverflowError`，并指向具体的方法调用栈（如`Exception in thread "main" java.lang.StackOverflowError`）。 | 日志提示`OutOfMemoryError`，可能附带具体原因（如`Java heap space`表示堆内存不足；`Direct buffer memory`表示直接内存不足）。                    |
| ​**解决方法**​   | - 减少递归深度（改为迭代）；  <br>- 优化方法调用链（拆分复杂方法）；  <br>- 增大JVM栈内存（`-Xss`参数，如`-Xss2m`增大栈大小）。                   | - 优化内存使用（释放无用对象、避免泄漏）；  <br>- 压缩对象（如图片加载时限制尺寸）；  <br>- 增大堆内存（`-Xmx`参数）；  <br>- 使用对象池复用对象。                     |
#### 二、详细对比与示例
#####  ​1.**触发位置与内存模型**​
- ​**栈内存（Stack）​**​：  
    每个线程独立拥有栈空间，用于存储方法调用的**栈帧**​（每个方法调用对应一个栈帧，包含局部变量表、操作数栈、返回地址等）。栈内存容量较小（默认通常为`1MB~几MB`），且遵循**后进先出（LIFO）​**原则。  
    ​**栈溢出**​：当方法调用次数过多（如无限递归），栈帧数量超过栈的最大容量时，无法为新方法调用分配栈帧，触发`StackOverflowError`。
    
- ​**堆内存（Heap）​**​：  
    所有线程共享的堆空间，用于存储对象实例（如`new Bitmap()`创建的对象）。堆内存容量较大（通常为几十MB到几百MB，可通过`-Xmx`调整）。  
    ​`OOM`**​：当对象实例持续创建且未被`GC`回收，堆内存被占满时，无法为新对象分配内存，触发`OutOfMemoryError`。
##### 2. ​**典型场景与示例**​
​**`StackOverflowError` 示例
```java
// 无限递归导致栈溢出
public class StackOverflowDemo {
    public static void main(String[] args) {
        recursiveMethod(0); // 调用递归方法
    }

    private static void recursiveMethod(int count) {
        System.out.println("Count: " + count);
        recursiveMethod(count + 1); // 无限递归，无终止条件
    }
}
```
**输出日志**​：
```
Count: 0  
Count: 1  
...（重复数千次后）  
Exception in thread "main" java.lang.StackOverflowError  
    at StackOverflowDemo.recursiveMethod(StackOverflowDemo.java:10)  
    at StackOverflowDemo.recursiveMethod(StackOverflowDemo.java:10)  
    ...（重复栈帧，最终指向递归方法）  
```
`OOM`示例
```java
// 加载超大Bitmap导致OOM
public class OomDemo extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 加载10000x10000像素的图片（未压缩）
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.large_image);
        // 若large_image的分辨率远超ImageView需求，且未压缩，直接导致OOM
    }
}
```
输出日志：
```
E/AndroidRuntime: FATAL EXCEPTION: main  
    Process: com.example.oomdemo, PID: 12345  
    java.lang.OutOfMemoryError: Failed to allocate a 400000012 byte allocation with 16777216 free bytes and 123MB until OOM  
        at dalvik.system.VMRuntime.newNonMovableArray(Native Method)  
        at android.graphics.BitmapFactory.nativeDecodeAsset(Native Method)  
        at android.graphics.BitmapFactory.decodeStream(BitmapFactory.java:818)  
        ...（指向BitmapFactory.decodeResource调用）  
```
##### 3. **解决方法对比**
|**错误类型**​|​**关键解决思路**​|​**具体措施**​|
|---|---|---|
|​**StackOverflowError**​|减少栈帧数量，避免栈空间耗尽。|- 无限递归→改为迭代（如用`for`循环代替递归）；  <br>- 深层调用→拆分复杂方法（减少单次调用栈深度）；  <br>- 增大栈内存→JVM参数`-Xss2m`（增大栈大小至2MB）。|
|​**OutOfMemoryError**​|减少堆内存占用，避免对象无法回收。|- 释放无用对象→及时关闭`Cursor`、`InputStream`，避免静态变量持有大对象；  <br>- 优化对象创建→循环中复用对象（如`StringBuilder`），使用对象池（如`LruCache`）；  <br>- 增大堆内存→JVM参数`-Xmx2g`（增大堆大小至2GB）；  <br>- 压缩对象→图片加载时限制尺寸（如Glide的`override(500,500)`）。|
#### **总结**​
- ​`StackOverflowError`**是**栈空间不足**，由方法调用层次过深导致，解决核心是减少栈帧数量；
- ​`OOM`**是**堆空间不足**，由对象过多或大对象未释放导致，解决核心是优化内存使用；
- 日志和内存模型是区分两者的关键（栈溢出指向方法调用栈，`OOM`指向对象分配失败）。