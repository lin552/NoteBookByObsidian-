---
创建时间: 2025-06-05 23:52:53
作者: wangxiaoming
tags:
  - Java
---

Java 泛型（Generics）是 Java 5 引入的核心特性，用于实现**类型安全**和**代码复用**，避免运行时类型转换错误。其考点覆盖**泛型语法**、**类型擦除**、**通配符**、**上下界限定**、**PECS 原则**、**类型推断**及**常见问题**等。以下是详细梳理：

#### 一、泛型基础语法
##### ​1. 泛型类、接口、方法​
- ​**泛型类**​：类名后声明类型参数（如 `class Box<T>`），使用时指定具体类型（如 `Box<String>`）。
    ```java
    class Box<T> {
        private T content;
        public void set(T content) { this.content = content; }
        public T get() { return content; }
    }
    Box<String> stringBox = new Box<>(); // 类型参数为 String
    ```
    
- ​**泛型接口**​：接口名后声明类型参数（如 `interface Comparable<T>`）。
    ```java
    interface Comparable<T> {
        int compareTo(T other);
    }
    class StringComparable implements Comparable<String> {
        @Override
        public int compareTo(String other) { /* 实现 */ }
    }
    ```
    
- ​**泛型方法**​：方法签名前声明类型参数（如 `public <T> void print(T t)`），可在静态方法中使用。
    ```java
    public class Utils {
        public static <T> void print(T t) {
            System.out.println(t);
        }
    }
    Utils.print("Hello"); // 自动推断 T 为 String
    ```

#### 二、类型擦除（Type Erasure）
Java 泛型是**编译期特性**，运行时类型参数会被擦除（Type Erasure），仅保留原始类型（Raw Type）。核心规则：
##### 1. 擦除规则​
- ​**无界类型参数**​（如 `T`）：擦除为 `Object`。
- ​**有界类型参数**​（如 `T extends Number`）：擦除为边界类型（`Number`）。
- ​**泛型方法**​：类型参数擦除后，方法参数类型替换为边界类型。

##### ​2. 擦除的影响​
- ​**运行时无法获取具体类型参数**​：`getClass()` 或反射无法得到 `T` 的具体类型（如 `new Box<String>().getClass()` 返回 `Box`）。
- ​**原始类型（Raw Type）​**​：未指定类型参数的泛型类（如 `Box`），编译时会警告，运行时可能引发 `ClassCastException`。

#### 三、通配符（Wildcard）
通配符 `?` 用于表示未知类型，解决泛型类型无法协变（Covariance）的问题。
##### ​1. 无界通配符（Unbounded Wildcard）​​
- ​**语法**​：`?`，表示任意类型。
- ​**用途**​：接受所有类型的泛型实例（如 `List<?>` 可接收 `List<String>`、`List<Integer>` 等）。
- ​**限制**​：不能添加除 `null` 外的元素（编译期无法确定类型）。

    ```java
    void printList(List<?> list) {
        for (Object obj : list) {
            System.out.println(obj); // 只能读取为 Object
        }
        // list.add("Hello"); // 编译错误：无法确定类型
    }
    ```

##### ​2. 上界通配符（Upper Bounded Wildcard）​​
- ​**语法**​：`? extends T`，表示类型为 `T` 或其子类型。
- ​**用途**​：限制泛型参数的上界，允许读取 `T` 类型数据（PECS 中的“Producer”）。
    ```java
    // 读取 List 中的 Number 类型数据（子类型如 Integer、Double 均可）
    double sum(List<? extends Number> list) {
        double total = 0.0;
        for (Number num : list) {
            total += num.doubleValue();
        }
        return total;
    }
    sum(Arrays.asList(1, 2.5, 3)); // 合法（Integer、Double 是 Number 子类型）
    ```
    
##### ​3. 下界通配符（Lower Bounded Wildcard）​​
- ​**语法**​：`? super T`，表示类型为 `T` 或其父类型。
- ​**用途**​：限制泛型参数的下界，允许写入 `T` 类型数据（PECS 中的“Consumer”）。
    ```java
    // 向 List 中写入 Integer 类型数据（父类型如 Number、Object 均可）
    void addIntegers(List<? super Integer> list) {
        for (int i = 1; i <= 5; i++) {
            list.add(i); // 只能写入 Integer 或其子类型（此处 Integer 是最终类）
        }
    }
    addIntegers(new ArrayList<Number>()); // 合法（Number 是 Integer 父类型）
    ```

#### 四、`PESC`原则（Producer Extends,Consumer Super）
PECS 是使用通配符的核心原则，用于解决泛型协变与逆变问题：

- ​**Producer（生产者）​**​：泛型对象**提供数据**​（只读），使用 `? extends T`（上界通配符）。  
    例如：`List<? extends Number>` 作为数据源，只能读取 `Number` 类型数据。
    
- ​**Consumer（消费者）​**​：泛型对象**接收数据**​（只写），使用 `? super T`（下界通配符）。  
    例如：`List<? super Integer>` 作为数据接收器，只能写入 `Integer` 类型数据。

#### 五、类型推断（Type Inference）
Java 编译器可根据上下文自动推断泛型类型参数（Type Argument），常见场景：

##### 1. 方法调用推断​
- ​**规则**​：根据方法参数的类型推断泛型类型。
    ```java
    // 推断 T 为 String（参数为 String）
    Utils.print("Hello"); // T 被推断为 String
    
    // 推断 T 为 Number（参数为 Integer 和 Double）
    List<Number> list = new ArrayList<>();
    list.add(1); // Integer → 推断 T 为 Number
    list.add(2.5); // Double → 推断 T 为 Number
    ```

#### ​**2. 变量声明推断**​
- ​**菱形运算符（Diamond Operator）​**​：`<>` 省略类型参数，编译器根据初始化值推断。
    ```java
    // 推断 T 为 String（初始化值为 "Hello"）
    Box<String> box = new Box<>(); 
    ```

#### 六、常见问题与注意事项
##### ​1. 泛型数组的限制​
- ​**无法直接创建泛型数组**​：`new T[10]` 编译错误（类型擦除后无法确定数组元素类型）。
- ​**替代方案**​：使用 `List<T>` 或反射绕过限制（但需注意类型安全）。
    ```java
    // 错误：无法创建泛型数组
    // T[] array = new T[10];
    
    // 正确：使用 List
    List<T> list = new ArrayList<>();
    ```
##### ​2. 原始类型（Raw Type）的风险​
- ​**定义**​：未指定类型参数的泛型类（如 `Box` 而非 `Box<String>`）。
- ​**风险**​：编译时类型检查失效，运行时可能抛出 `ClassCastException`。
    ```java
    Box rawBox = new Box();
    rawBox.set(123); // 存入 Integer
    String str = (String) rawBox.get(); // 运行时 ClassCastException
    ```
    
#### ​**3. 泛型与反射的兼容性**​
- ​**类型信息丢失**​：反射无法获取泛型参数的具体类型（如 `Field.getType()` 返回原始类型）。
- ​**解决方案**​：通过 `ParameterizedType` 获取父类的泛型参数（如 `MyClass<String>` 的父类泛型）。

#### 七、高频面试题
1. **Java 泛型的作用是什么？​**​  
    保证类型安全，避免运行时类型转换错误；提高代码复用性（如集合类）。
    
2. ​**类型擦除是什么？如何影响泛型？​**​  
    编译期擦除泛型类型参数，运行时仅保留原始类型。导致无法获取具体类型信息，但保证了二进制兼容性。
    
3. ​**通配符 `?`、`? extends T`、`? super T` 的区别是什么？​**​  
    `?` 表示任意类型；`? extends T` 表示 T 或其子类型（Producer）；`? super T` 表示 T 或其父类型（Consumer）。
    
4. ​**PECS 原则是什么？举例说明。​**​  
    Producer 用 `extends`（只读），Consumer 用 `super`（只写）。例如：`List<? extends Number>` 作为数据源（读取 Number），`List<? super Integer>` 作为数据接收器（写入 Integer）。
    
5. ​**为什么不能创建泛型数组？如何解决？​**​  
    类型擦除导致无法确定数组元素类型。解决方案：使用 `List<T>` 或反射（需谨慎处理类型安全）。
    
6. ​**泛型方法如何声明？类型推断的规则是什么？​**​  
    方法签名前声明类型参数（如 `public <T> void method(T t)`）。类型推断根据上下文自动确定 T 的具体类型。