---
创建时间: 2025-05-21 17:10:28
作者: wangxiaoming
tags:
  - 设计模式
  - 工厂模式
  - 创建型
---
#### 一、简单工厂 (Simple Factory)

​**特点**​：通过条件分支创建对象，非 `GoF` 官方模式，但简单实用。  
​**缺点**​：违反开闭原则（新增类型需修改工厂类）。

```java
interface Shape {
    void draw();
}

class Circle implements Shape { /* ... */ }
class Square implements Shape { /* ... */ }

class SimpleFactory {
    public static Shape createShape(String type) {
        if ("circle".equals(type)) return new Circle();
        else if ("square".equals(type)) return new Square();
        throw new IllegalArgumentException("Unknown type");
    }
}
```

#### 二、工厂方法（Factory Method）
​**特点**​：定义工厂接口，由子类决定实例化哪个类，符合开闭原则。  
​**适用场景**​：对象创建逻辑复杂或需要动态扩展。

```java
interface ShapeFactory {
    Shape createShape();
}

class CircleFactory implements ShapeFactory {
    @Override
    public Shape createShape() { return new Circle(); }
}

class SquareFactory implements ShapeFactory {
    @Override
    public Shape createShape() { return new Square(); }
}
```

#### 三、抽象工厂 （Abstract Factory）
**特点**​：创建**产品族**​（相关对象的集合），而非单一产品。  
​**适用场景**​：跨产品线的对象创建，如主题切换（黑暗模式/明亮模式）。

```java
// 抽象产品：按钮、文本框
interface Button { void render(); }
interface TextBox { void input(); }

// 具体产品：黑暗模式
class DarkButton implements Button { /* ... */ }
class DarkTextBox implements TextBox { /* ... */ }

// 抽象工厂：主题工厂
interface ThemeFactory {
    Button createButton();
    TextBox createTextBox();
}

// 具体工厂：黑暗主题
class DarkThemeFactory implements ThemeFactory {
    @Override
    public Button createButton() { return new DarkButton(); }
    @Override
    public TextBox createTextBox() { return new DarkTextBox(); }
}
```