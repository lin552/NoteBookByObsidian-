---
创建时间: "2025-06-24 14:30:25"
作者: wangxiaoming
tags:
---

在 Java 中，类的初始化顺序遵循严格的规则，涉及**类加载阶段**和**实例创建阶段**的不同成员初始化顺序。理解这些规则对避免程序逻辑错误（如空指针、初始化顺序导致的 bug）至关重要。以下是完整的初始化顺序解析：

#### 一、类初始化的触发条件
类的初始化（`<clinit>()` 方法执行）发生在**类被主动使用**时，常见触发场景包括：
- 创建类的实例（`new ClassName()`）；
- 调用类的静态方法（`ClassName.staticMethod()`）；
- 访问类的静态变量（非 `final` 常量）；
- 反射调用类（`Class.forName("ClassName")`）；
- 初始化子类（触发父类初始化）；
- 主类（`main` 方法所在类）。

#### 二、类初始化的核心顺序（类加载阶段）
类的初始化阶段（`<clinit>()` 方法）负责执行**静态成员的初始化**，其顺序严格遵循以下规则：
##### 1. ​**父类优先于子类**​
子类的初始化会**先触发父类的初始化**​（递归向上，直到 `Object` 类）。这是因为 Java 要求子类必须能访问父类的静态成员，因此父类必须先完成初始化。
##### 2. ​**静态变量与静态代码块按定义顺序执行**​
在同一个类中，静态变量（`static` 字段）和静态代码块（`static {}`）的执行顺序**严格按照它们在类中定义的顺序**。
```java
public class Parent {
    static int a = initA(); // 步骤1：初始化a
    static { 
        System.out.println("Parent 静态代码块1，a=" + a); // 步骤2：输出a的值（已初始化）
    }
    static int b = initB(); // 步骤3：初始化b
    static { 
        System.out.println("Parent 静态代码块2，b=" + b); // 步骤4：输出b的值（已初始化）
    }

    private static int initA() {
        System.out.println("初始化Parent.a");
        return 1;
    }
    private static int initB() {
        System.out.println("初始化Parent.b");
        return 2;
    }
}
```
输出结果
```
初始化Parent.a
Parent 静态代码块1，a=1
初始化Parent.b
Parent 静态代码块2，b=2
```

#### 三、实例初始化的顺序（对象创建阶段）
当创建类的实例（`new ClassName()`）时，实例的初始化顺序在类初始化之后执行，遵循以下规则：
##### 1. ​**父类实例初始化先于子类**​
子类实例的初始化会**先触发父类实例的初始化**​（递归向上，直到 `Object` 类）。
##### 2. ​**实例变量与实例代码块按定义顺序执行**​
在同一个类中，实例变量（非 `static` 字段）和实例代码块（`{}`）的执行顺序**严格按照它们在类中定义的顺序**，且在构造函数之前执行。
##### 3. ​**构造函数最后执行**​
构造函数是实例初始化的最后一步，用于完成对象的最终状态设置。

#### 四、完整初始化顺序总结（父类->子类）
以下是一个完整的初始化顺序示例（父类 `Parent` 和子类 `Child`）：
```java
public class Parent {
    // 静态变量（类初始化阶段）
    static int staticParentVar = initStaticParentVar();
    static { 
        System.out.println("Parent 静态代码块，staticParentVar=" + staticParentVar); 
    }

    // 实例变量（实例初始化阶段）
    int instanceParentVar = initInstanceParentVar();
    { 
        System.out.println("Parent 实例代码块，instanceParentVar=" + instanceParentVar); 
    }

    // 构造函数（实例初始化阶段最后）
    public Parent() {
        System.out.println("Parent 构造函数");
    }

    private static int initStaticParentVar() {
        System.out.println("初始化 Parent 静态变量");
        return 10;
    }
    private int initInstanceParentVar() {
        System.out.println("初始化 Parent 实例变量");
        return 20;
    }
}

public class Child extends Parent {
    // 静态变量（类初始化阶段）
    static int staticChildVar = initStaticChildVar();
    static { 
        System.out.println("Child 静态代码块，staticChildVar=" + staticChildVar); 
    }

    // 实例变量（实例初始化阶段）
    int instanceChildVar = initInstanceChildVar();
    { 
        System.out.println("Child 实例代码块，instanceChildVar=" + instanceChildVar); 
    }

    // 构造函数（实例初始化阶段最后）
    public Child() {
        System.out.println("Child 构造函数");
    }

    private static int initStaticChildVar() {
        System.out.println("初始化 Child 静态变量");
        return 30;
    }
    private int initInstanceChildVar() {
        System.out.println("初始化 Child 实例变量");
        return 40;
    }

    public static void main(String[] args) {
        new Child(); // 触发 Child 和 Parent 的初始化
    }
}
```
输出结果：
```
初始化 Parent 静态变量          // 父类静态变量（类初始化阶段）
Parent 静态代码块，staticParentVar=10  // 父类静态代码块（类初始化阶段）
初始化 Child 静态变量           // 子类静态变量（类初始化阶段）
Child 静态代码块，staticChildVar=30    // 子类静态代码块（类初始化阶段）
初始化 Parent 实例变量          // 父类实例变量（实例初始化阶段）
Parent 实例代码块，instanceParentVar=20  // 父类实例代码块（实例初始化阶段）
Parent 构造函数                // 父类构造函数（实例初始化阶段）
初始化 Child 实例变量           // 子类实例变量（实例初始化阶段）
Child 实例代码块，instanceChildVar=40    // 子类实例代码块（实例初始化阶段）
Child 构造函数                // 子类构造函数（实例初始化阶段）
```
#### 五、特殊情况说明
##### 1. ​**静态内部类的初始化**​
静态内部类（`static class Inner`）的初始化**不会触发外部类的初始化**，只有在首次使用内部类时（如创建实例、访问静态成员）才会触发自身的初始化。

##### 2. ​`final` 变量的特殊处理**​
- ​**编译时常量**​（`static final` 且在编译时确定值）：不会触发类的初始化，因为其值会被内联到使用它的代码中。  
    示例：`static final int MAX = 100;`（编译时已知）。
- ​**运行时常量**​（`static final` 但在运行时确定值）：会触发类的初始化。  
    示例：`static final int RANDOM = new Random().nextInt(100);`（运行时赋值）。

#### 总结
Java 类的初始化顺序可总结为以下优先级：
**父类静态成员（变量→代码块）→ 子类静态成员（变量→代码块）→ 父类实例成员（变量→代码块）→ 父类构造函数 → 子类实例成员（变量→代码块）→ 子类构造函数**。