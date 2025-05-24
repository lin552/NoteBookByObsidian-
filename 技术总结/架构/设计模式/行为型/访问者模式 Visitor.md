---
创建时间: 2025-05-21 18:40:18
作者: wangxiaoming
tags:
  - 设计模式
  - 访问者模式
  - 行为型
---
- ​**核心思想**​：在不修改类结构的前提下，为类添加新操作。
- ​**场景**​：编译器设计（`AST` 遍历）、对象序列化。

```java
interface Visitor {
    void visit(ElementA e);
    void visit(ElementB e);
}

abstract class Element {
    abstract void accept(Visitor visitor);
}

class ElementA extends Element {
    @Override
    void accept(Visitor v) { v.visit(this); }
}

class ElementB extends Element {
    @Override
    void accept(Visitor v) { v.visit(this); }
}
```
