---
创建时间: 2025-05-21 21:47:28
作者: wangxiaoming
tags:
  - Java
  - 类加载器
  - 双亲委派
---
#### 一、类加载流程
1. ​**加载（Loading）​**​
    - 通过类的全限定名获取二进制字节流（如 `.class` 文件、网络、动态生成等）。
    - 将字节流转换为方法区内的运行时数据结构。
    - 在堆中生成一个 `Class` 对象，作为方法区数据的访问入口。
2. ​**验证（Verification）​**​
    - 确保字节码符合 `JVM` 规范，防止恶意代码破坏 `JVM`。
    - 包括文件格式验证、元数据验证、字节码验证、符号引用验证。
3. ​**准备（Preparation）​**​
    - 为类的静态变量分配内存并设置默认初始值（如 `int` 初始化为 `0`）。
    - 如果静态变量是常量（`final static`），则直接赋值为代码中的值。
4. ​**解析（Resolution）​**​
    - 将常量池中的符号引用（如类名、方法名）转换为直接引用（内存地址）。
5. ​**初始化（Initialization）​**​
    - 执行类的静态初始化块（`static{}`）和静态变量的显式赋值。
    - 由 `CLINIT` 方法完成，由`JVM` 保证线程安全。

#### 二、类加载器（`ClassLoader`）
##### ​**1. 双亲委派模型**​
- ​**工作流程**​：  
    当加载一个类时，先委托父类加载器尝试加载；若父类无法加载（如找不到），子类才尝试自己加载。
- ​**核心类加载器**​：
    1. ​`Bootstrap ClassLoader`​
        - 负责加载 `JRE/lib` 下的核心类（如 `java.lang.*`）。
        - 用 C++ 实现，是 `JVM` 自身的一部分。
    2. ​`Extension ClassLoader`​
        - 加载 `JRE/lib/ext` 下的扩展类。
    3. ​`Application ClassLoader`​
        - 加载用户类路径（`classpath`）下的类。
- ​**作用**​：避免重复加载核心类，防止用户自定义类覆盖核心类（如自定义 `java.lang.Object`）。
##### ​**2. 自定义类加载器**​
- 继承 `ClassLoader`，重写 `findClass` 方法（不要破坏双亲委派）。
- 示例代码：
```java
public class CustomClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name); // 从自定义来源加载字节码
        return defineClass(name, classData, 0, classData.length);
    }
}
```
##### ​**3. 打破双亲委派**​
- 重写 `loadClass` 方法（慎用，可能导致安全问题）。
- Tomcat 的 `WebAppClassLoader` 为隔离 Web 应用打破双亲委派。

#### 三、类初始化时机
- 显式触发：`Class.forName("ClassName", true, classLoader)`（第二个参数为 `true` 时初始化）。
- 隐式触发：
    - 创建类的实例（`new`）。
    - 访问类的静态变量或静态方法。
    - 反射调用 `newInstance()`（`JDK 9+ 已废弃`）。
- ​**不会触发初始化的情况**​：
    - 通过子类引用父类的静态变量。
    - 定义数组（如 `ClassA[] arr = new ClassA[10]`）。
    - 访问 `final static` 常量（编译期常量）。
#### 四、常见问题与解决
1. ​**`ClassNotFoundException`**​
    - 原因：类未在类路径中找到。
    - 解决：检查类路径配置或依赖引入。
2. ​**`NoClassDefFoundError`**​
    - 原因：类在编译时存在，但运行时缺失。
    - 解决：检查依赖冲突或缺失。
3. ​**类型转换异常（`ClassCastException`）​**​
    - 原因：同一类被不同类加载器加载，视为不同类。
    - 解决：统一类加载器。
#### 五、高级特性
- ​**热部署**​：通过自定义类加载器重新加载修改后的类（需避免内存泄漏）。
- ​**模块化系统（`JPMS`）​**​：通过 `module-info.java` 控制类可见性，影响类加载行为。
- `​OSGi` 框架**​：动态模块化系统，每个模块有独立类加载器。